def secrets = [
    [path: params.VAULT_PATH_SVC_ACCOUNT_EPHEMERAL, engineVersion: 1, secretValues: [
        [envVar: 'OC_LOGIN_TOKEN_DEV', vaultKey: 'oc-login-token-dev'],
        [envVar: 'OC_LOGIN_SERVER_DEV', vaultKey: 'oc-login-server-dev']]],
    [path: params.VAULT_PATH_QUAY_PUSH, engineVersion: 1, secretValues: [
        [envVar: 'QUAY_USER', vaultKey: 'user'],
        [envVar: 'QUAY_TOKEN', vaultKey: 'token']]],
    [path: params.VAULT_PATH_RHR_PULL, engineVersion: 1, secretValues: [
        [envVar: 'RH_REGISTRY_USER', vaultKey: 'user'],
        [envVar: 'RH_REGISTRY_TOKEN', vaultKey: 'token']]]
]

def configuration = [vaultUrl: params.VAULT_ADDRESS, vaultCredentialId: params.VAULT_CREDS_ID, engineVersion: 1]

pipeline {
    agent { label 'rhel8' }
    options {
        timestamps()
    }
    environment {
        // --------------------------------------------
        // Options that must be configured by app owner
        // --------------------------------------------
        APP_NAME="rbac"  // name of app-sre "application" folder this component lives in
        COMPONENT_NAME="spicedb-operator"  // name of app-sre "resourceTemplate" in deploy.yaml for this component
        IMAGE="quay.io/cloudservices/spicedb-operator"  // image location on quay
        RUN_PLATSEC=true // optional step to run vulnerability checks
        IQE_PLUGINS="CHANGEME"  // name of the IQE plugin for this app.
        IQE_MARKER_EXPRESSION="CHANGEME"  // This is the value passed to pytest -m
        IQE_FILTER_EXPRESSION=""  // This is the value passed to pytest -k
        IQE_CJI_TIMEOUT="30m"  // This is the time to wait for smoke test to complete or fail

        CICD_URL="https://raw.githubusercontent.com/RedHatInsights/cicd-tools/main"
    }
    stages {
        stage('Build the PR commit image') {
            steps {
                withVault([configuration: configuration, vaultSecrets: secrets]) {
                    sh './build_deploy.sh'
                }
                
                sh 'mkdir -p artifacts'
            }
        }

        // stage('Run Tests') {
        //     parallel {
        //         stage('Run unit tests') {
        //             steps {
        //                 withVault([configuration: configuration, vaultSecrets: secrets]) {
        //                     sh 'bash -x ./scripts/unit_test.sh'
        //                 }
        //             }
        //         }

        //         stage('Run smoke tests') {
        //             steps {
        //                 withVault([configuration: configuration, vaultSecrets: secrets]) {
        //                     sh '''
        //                         curl -s $CICD_URL/bootstrap.sh > .cicd_bootstrap.sh
                                
        //                         source ./.cicd_bootstrap.sh
        //                         source "${CICD_ROOT}/deploy_ephemeral_env.sh"
        //                         source "${CICD_ROOT}/cji_smoke_test.sh"
        //                     '''
        //                 }

        //             }
        //         }
                
        //         stage ('Run vulnerability tests') {
        //             if (env.RUN_PLATSEC == true)
        //                 steps {
        //                     withVault([configuration, vaultSecrets: secrets]) {
        //                         sh 'source ${CICD_ROOT}/platsec.sh "$IMAGE" "$ARTIFACTS_DIR"' 
        //                 }
        //             }
        //         }   
        //     }
        // }
    }

    post {
        always{
            withVault([configuration: configuration, vaultSecrets: secrets]) {
                sh '''
                    curl -s $CICD_URL/bootstrap.sh > .cicd_bootstrap.sh
                    source ./.cicd_bootstrap.sh

                    source "${CICD_ROOT}/post_test_results.sh"
                '''
            }
        }
    }
}