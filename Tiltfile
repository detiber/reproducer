# Tilt >= v0.17.8 is required to handle escaping of colons in selector names and proper teardown of resources
load('ext://min_tilt_version', 'min_tilt_version')
min_tilt_version('0.17.8')

# We require at minimum CRD support, so need at least Kubernetes v1.16
load('ext://min_k8s_version', 'min_k8s_version')
min_k8s_version('1.16')

k8s_yaml('cluster-network-addons.yaml')
k8s_resource(
    workload='cluster-network-addons-operator',
    objects=[
        'cluster-network-addons:namespace',
        'networkaddonsconfigs.networkaddonsoperator.network.kubevirt.io:customresourcedefinition',
        'cluster-network-addons-operator:serviceaccount',
        'cluster-network-addons-operator:role',
        'cluster-network-addons-operator:clusterrole',
        'cluster-network-addons-operator:rolebinding',
        'cluster-network-addons-operator:clusterrolebinding',
    ],
)

k8s_yaml('nac.yaml')
k8s_resource(
    new_name='cluster-network-addons',
    objects=[
        'cluster:networkaddonsconfig'
    ],
    resource_deps=['cluster-network-addons-operator'],
)
    
local_resource('wait-for-networkaddonsconfig', 
    cmd='kubectl wait --for condition=available  networkaddonsconfigs/cluster --timeout=60s', 
    resource_deps=['cluster-network-addons'] 
)

# workaround since we can't override the macvtap-cni config when using hte cluster-network-addons-operator
k8s_yaml('macvtap-cni.yaml')
k8s_yaml('macvtap-cni-config.yaml')
k8s_resource(
    workload='macvtap-cni',
    objects=[
        'macvtap-deviceplugin-config:configmap'
    ]
)

k8s_yaml('macvtap-nad.yaml')
k8s_resource(
    new_name='macvtap-nad',
    objects=['tink:networkattachmentdefinition'],
    resource_deps=['wait-for-networkaddonsconfig']
)

# KubeVirt
k8s_yaml('kubevirt-operator.yaml')
k8s_resource(
    workload='virt-operator',
    objects=[
        'kubevirt:namespace',
        'kubevirts.kubevirt.io:customresourcedefinition',
        'kubevirt-operator:serviceaccount',
        'kubevirt-operator:role',
        'kubevirt-operator:clusterrole',
        'kubevirt-operator-rolebinding:rolebinding',
        'kubevirt-operator:clusterrolebinding',
        'kubevirt-cluster-critical:priorityclass',
        'kubevirt.io\\:operator:clusterrole'
    ],
    resource_deps=['wait-for-networkaddonsconfig']
)

kubevirt_config = {
    'apiVersion': 'v1',
    'kind': 'ConfigMap',
    'metadata': {
        'name': 'kubevirt-config',
        'namespace': 'kubevirt'
    },
    'data': {
        'feature-gates': 'Macvtap'
    }
}
k8s_yaml(encode_yaml(kubevirt_config))

kubevirt_cr = {
    'apiVersion': 'kubevirt.io/v1alpha3',
    'kind': 'KubeVirt',
    'metadata': {
        'name': 'kubevirt',
        'namespace': 'kubevirt'
    },
    'spec': {
        'certificateRotateStrategy': {},
        'configuration': {},
        'customizeComponents': {},
        'imagePullPolicy': 'IfNotPresent',
    }
}
k8s_yaml(encode_yaml(kubevirt_cr))
k8s_resource(
    new_name='kubevirt',
    objects=[
        'kubevirt:kubevirt',
        'kubevirt-config:configmap:kubevirt'
    ],
    resource_deps=['virt-operator']    
)
