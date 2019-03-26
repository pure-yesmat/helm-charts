@Library('pso-jenkins-library') _

//////////////////////////////////////////////////////////////////////////////
// Helper functions

def internalScm = new pso.pipeline.utils.scm()
def nsmDev = new pso.pipeline.utils.nsmdev()

class Constants {
    static final String NSM_ENV_VERSION = "4.0.1-b409a067"
    static final String NSM_ENV_IMAGE = "pc2-dtr.dev.purestorage.com/purestorage/nsm-toolbox"
    static final String GO_REPO_ROOT = "/go/src/pso.purestorage.com/nsm"
    static final String NODE_LABEL_NSTK_LP_DEFAULT = "newstack-launchpad"
    static final String NODE_LABEL_NSTK_LP_FC = "newstack-launchpad-fc"
}

// Required due to JENKINS-27421
@NonCPS
List<List<?>> mapToList(Map map) {
    return map.collect { it ->
        [it.key, it.value]
    }
}

def getParametersForTestStage(LinkedHashMap tests, String stageName, String stageDescription) {
    def params = []
    params.add(booleanParam(defaultValue: true, description: "${stageDescription}", name: "${stageName}"))
    for (kv in mapToList(tests)) {
        params.add(booleanParam(defaultValue: true, description: "(${stageName}) Run ${kv[0]}", name: "${kv[0]}"))
    }
    return params
}

/////////////////////////////////////////////////////////////////////////////////
// Actually run the tasks

def k8sTestCheckout = {
    dir("helm-repo") {
        internalScm.checkoutSCMWithTags(scm)
    }

    dir("k8s-repo") {
        internalScm.checkOutFromInternal("k8s", "master")
    }
}

def k8sTests = [:]
nsmDev.addTest(Constants.NODE_LABEL_NSTK_LP_DEFAULT, k8sTests, "TestK8S", 15, k8sTestCheckout) {
    sh "pwd"
    sh "ls -l"
    sh "ls -l helm-repo/"
    sh "ls -l k8s-repo/"
    sh 'ls -l ${NSM_SRC_DIR}'
    sh "ls -l ${Constants.GO_REPO_ROOT}"
}

//////////////////////////////////////////////////////////////////////////////
// Setup some properties/parameters for the build

def parametersToAdd = getParametersForTestStage(k8sTests, "K8S", 'Run K8S Stage')
properties([
        parameters(parametersToAdd),
])

if (params["K8S"]) {
    stage("K8S") {
        parallel k8sTests
    }
}
