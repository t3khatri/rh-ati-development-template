properties([
    parameters([
        choice(name: 'TARGET_CLOUDHUB_ENVIRONMENT', choices: ['DEV1','INTEGRATION1','INTEGRATION2','INTEGRATION3', 'DEV2','DEV3','QA1','QA2','QA3','PRODMIRROR','PRODUCTION']),
        choice(name: 'ANYPOINT_BUSINESS_GROUP',     choices: ['ati','hrfs','ms','sfcc']),
        choice(name: 'CLOUDHUB_WORKERS',            choices: ['1','2','3','4','5','6','7','8']),
        choice(name: 'CLOUDHUB_WORKER_TYPE',        choices: ['MICRO(0.1 vCores)','SMALL(0.2 vCores)','MEDIUM(1 vCore )','LARGE(2 vCores)','XLARGE(4 vCores)','XXLARGE(8 vCores)','4XLARGE(16 vCores)']),

        // API promotion
        choice(name: 'ASSET_TYPE',                  choices: ['Integration','API'], description: 'Type of  Mule Application (e.g. API or Integration)'),
        string(name: 'CLIENT_APP',                  description: 'Name of the Client Application for API Promotion'),
        string(name: 'SOURCE_CLOUDHUB_ENVIRONMENT', description: 'Source environment from which API to be promoted'),
        string(name: 'VERSION_NUMBER',              defaultValue: '', trim: true),
        string(name: 'API_VERSION',                 defaultValue: '1.0', description: 'Version of API Instance for pairing with Mule Application (e.g. 1.0)' ),
        string(name: 'API_ID',                      defaultValue: '111', description: 'API ID for API promotion' ),

        choice(name: 'MUNIT_READY',                 choices: ['N', 'Y'] ),
        choice(name: 'SONAR_READY',                 choices: ['N', 'Y'] ),

        choice(name: 'NEXUS_READY',                 choices: ['N', 'Y'] ),
        choice(name: 'RELEASE_READY',               choices: ['N', 'Y'] ),

        choice(name: 'AMQ_CREATION_READY',          choices: ['N', 'Y'] ),

        choice(name: 'DEPLOY_READY',                choices: ['N', 'Y'] ),
        string(name: 'RELEASE_VERSION',             defaultValue: '', description: 'Release Version to be deployed (e.g. 1.0.0)' )
    ])
])

pipeline {
    agent {
        label 'AmazonLinux2@itoss-tools-prd'
    }
    environment {
        BUILD_VERSION           = "1.0.${currentBuild.number}-SNAPSHOT"
        MVN_SET                 = credentials('mulesoft-commons-connected-app-settings-xml')
        CONNECTED_APP_CREDS     = credentials('mulesoft-connected-app-non-prod')

        SONAR_QUBE_CREDS        = credentials('mulesoft-sonar-qube-non-prod')
        SONAR_QUBE_SERVER       = "https://sonarqube-dev.rh-ccoe-dev.rhgcs.com/"

        // temp test, working Nexus
        NEXUS_SERVER            = "https://nexus.rhgcs.com/repository/rh-sfcc-maven-hosted"
        // provided by Syed, not working. TBD
        //NEXUS_SERVER            = "https://nexus.rhgcs.com/repository/rh-sfcc-maven-group/"

        NEXUS_ID                = "rh-nexus"

        // Move CLOUDHUB_REGION to parameters if this changes based on DLB/Deployment architecture discussion. TBD
        CLOUDHUB_REGION         = "us-west-1"
        // Move coverage % to parameters if MUNIT is mandated when CI-CD automation is live. TBD
        MUNIT_APP_COVERAGE      = 0
        MUNIT_RES_COVERAGE      = 0
        MUNIT_FLOW_COVERAGE     = 0

    }

    tools {
        jdk "openjdk-1.8.0"
        nodejs "nodejs-16.14.2"
        maven "maven-3.6.3"
    }

    stages {

/*         stage('TEST') {
            steps {
                script {
                    echo "Upload to Nexus"
                    sh "mvn package -s  $MVN_SET -DskipTests"
                    def pom = readMavenPom file: 'pom.xml'
                    sh """ mvn -DskipTests deploy:deploy-file \
                        -s  $MVN_SET \
                        -DgroupId=${pom.groupId} \
                        -DartifactId=${pom.artifactId} \
                        -Dversion=${pom.version} \
                        -DgeneratePom=true \
                        -Dpackaging=jar \
                        -DrepositoryId=rh-nexus \
                        -Durl=https://nexus.rhgcs.com/repository/rh-sfcc-maven-hosted \
                        -Dfile=./target/${pom.artifactId}-${pom.version}-mule-application.jar """
                    echo "Artifact Uploaded to Nexus: ${currentBuild.currentResult}"
                }
            }
            post {
                success {
                    echo "...Upload to Nexus is Succeeded for ${env.BUILD_VERSION}: ${currentBuild.currentResult}"
                }
                unsuccessful {
                    echo "...Upload to Nexus is Failed for ${env.BUILD_VERSION}: ${currentBuild.currentResult}"
                }
            }
        } */

        stage('Parameters validations') {
            steps {
                echo "Parameters validations started"
                script {

                        if ("${currentBuild.number}" == '1')
                        {
                            currentBuild.result = 'ABORTED'
                            error('Build aborted as this is the first build')
                        }
                        else if ("$BRANCH_NAME" == 'main') // || "${params.RELEASE_READY}" == 'Y'
                        {
                            currentBuild.result = 'ABORTED'
                            error('Build aborted as wrong value for RELEASE_READY selected')
                        }
                        else {
                            echo "...Parameters validations looks good for ${env.BUILD_VERSION}: ${currentBuild.currentResult}"
                            echo "TARGET_CLOUDHUB_ENVIRONMENT   = '${params.TARGET_CLOUDHUB_ENVIRONMENT} '"
                            echo "ANYPOINT_BUSINESS_GROUP       = '${params.ANYPOINT_BUSINESS_GROUP}'"
                            echo "CLOUDHUB_WORKERS              = '${params.CLOUDHUB_WORKERS}'"
                            echo "CLOUDHUB_WORKER_TYPE          = '${params.'CLOUDHUB_WORKER_TYPE'}'"
                            echo "MUNIT_READY                   = '${params.MUNIT_READY}'"
                            echo "RELEASE_READY                 = '${params.RELEASE_READY}'"
                            echo "SONAR_READY                   = '${params.SONAR_READY}'"
                            echo "NEXUS_READY                   = '${params.NEXUS_READY}'"
                            echo "AMQ_CREATION_READY            = '${params.AMQ_CREATION_READY}'"
                            echo "RELEASE_READY                 = '${params.RELEASE_READY}'"

                            echo "RELEASE_VERSION               = '${params.RELEASE_VERSION}'"
                            echo "DEPLOY_READY                  = '${params.DEPLOY_READY}'"
                            echo "REGRESSION_SUITE_READY        = '${params.REGRESSION_SUITE_READY}'"
                        }
                }
            }
            post {
                success {
                    echo "...Parameters validations Succeeded for ${env.BUILD_VERSION}: ${currentBuild.currentResult}"
                }
                unsuccessful {
                    echo "...Parameters validations Failed for ${env.BUILD_VERSION}: ${currentBuild.currentResult}"
                }
            }
        }

        stage('Build and MUnit Phase') {
            when {
                expression { params.MUNIT_READY == 'Y' }
            }
            steps {
                echo "Starting Build and Test..."
                sh "mvn clean test -s $MVN_SET -Dapp-coverage=$MUNIT_APP_COVERAGE -Dres-coverage=$MUNIT_RES_COVERAGE -Dflow-coverage=$MUNIT_FLOW_COVERAGE"
                echo "Build and Test: ${currentBuild.currentResult}"
            }
            post {
                success {
                    echo "...Build and Test Succeeded for ${env.BUILD_VERSION}: ${currentBuild.currentResult}"
                } 
                unsuccessful {
                    echo "...Build and Test Failed for ${env.BUILD_VERSION}: ${currentBuild.currentResult}"
                    script {
        	            notifyUserMunit()
                    }
                }
            }
        }

        stage('Sonar Code Scan') {
            when {
                expression { params.SONAR_READY == 'Y' }
            }
            tools {
               jdk "openjdk-1.11.0"
            }
            steps {
                echo 'Parameters validations - JAVA Version Check'
                sh 'java -version'
                echo "Starting Sonar Code Scan..."
                sh "mvn clean sonar:sonar -s $MVN_SET -Dsonar.host.url=$SONAR_QUBE_SERVER -Dsonar.sources=src/ -Dsonar.login=$SONAR_QUBE_CREDS_USR -Dsonar.password=$SONAR_QUBE_CREDS_PSW"
                echo "Sonar Code Scan: ${currentBuild.currentResult}"
            }
            post {
                success {
                    echo "...Sonar Code Scan Succeeded for ${env.BUILD_VERSION}: ${currentBuild.currentResult}"
                }
                unsuccessful {
                    echo "...Sonar Code Scan Failed for ${env.BUILD_VERSION}: ${currentBuild.currentResult}"
                    script {
        	            notifyUserMunit()
                    }
                }
            }
        }

        stage('Promote API') {
            when {
                expression { params.ASSET_TYPE == 'API' }
            }
            steps {
                script {
                    echo "Promoting API from Development..."
                    def pom = readMavenPom file: 'pom.xml'
                    print "POM Name: " + pom.name
                    print "POM artifactId: " + pom.artifactId
                    print "Mule Runtime: " + pom.properties['app.runtime']
                    sh """ newman run postman/ci-cd/promote-api.postman-collection.json \
                                --env-var anypoint_username='$CCONNECTED_APP_CREDS_USR' \
                                --env-var anypoint_password='$CONNECTED_APP_CREDS_PSW' \
                                --env-var anypoint_organisation='${params.ANYPOINT_BUSINESS_GROUP}' \
                                --env-var source_environment='${params.SOURCE_CLOUDHUB_ENVIRONMENT}' \
                                --env-var target_environment='${params.TARGET_CLOUDHUB_ENVIRONMENT}' \
                                --env-var asset_id=${pom.artifactId} \
                                --env-var product_version='${params.API_VERSION}' \
                                --env-var anypoint_runtime=${pom.properties['app.runtime']} \
                                --env-var client_app_name='${params.CLIENT_APP}' \
                                --disable-unicode \
                                --reporters cli,json \
                                --reporter-json-export promote-api-output.json """
                    echo "Promoted API from Testing: ${currentBuild.currentResult}"
                }
            }
            post {
                success {
                    echo "...Promote API from ${params.SOURCE_CLOUDHUB_ENVIRONMENT} environment to ${params.TARGET_CLOUDHUB_ENVIRONMENT} environment Succeeded for ${env.BUILD_VERSION}: ${currentBuild.currentResult}"
                }
                failure {
                    echo "...Promote API from ${params.SOURCE_CLOUDHUB_ENVIRONMENT} environment to ${params.TARGET_CLOUDHUB_ENVIRONMENT} environment Failed for ${env.BUILD_VERSION}: ${currentBuild.currentResult}"
                }
            }
        }

        stage('Anypoint MQ (AMQ) validation') {
            when {
                expression { params.AMQ_CREATION_READY == 'Y' }
            }
            steps {
                echo "Queue creation validation and creation"
                sh 'newman run postman/ci-cd/amq-api.postman-collection.json'
            }
            post {
                success {
                    echo "...Anypoint MQ (AMQ) validation is Succeeded for ${env.BUILD_VERSION}: ${currentBuild.currentResult}"
                }
                unsuccessful {
                    echo "...Anypoint MQ (AMQ) validation is Failed for ${env.BUILD_VERSION}: ${currentBuild.currentResult}"
                }
            }
        }

        stage('Upload to Nexus') {
            when {
                expression { params.NEXUS_READY == 'Y' }
            }
            steps {
                script {
                    def pom = readMavenPom file: 'pom.xml'
                    env.GROUP_ID = pom.groupId
                    env.ARTIFACT_ID = pom.artifactId
                    env.VERSION_TOBE_RELEASED = pom.version.split("\\-")[0]
                }
                // maven-settings-dev dev maven
                configFileProvider(
                    [configFile(fileId: 'sfcc-maven-settings', variable: 'MAVEN_SETTINGS_NEXUS_DEV')]) {
                    script {
                        echo "Upload to Nexus"
                        sh "mvn package -s $MVN_SET -DskipTests"
                        echo "NEXUS_ID = '${NEXUS_ID}'"
                        echo "GROUP_ID = '${GROUP_ID}'"

                        sh 'mvn deploy:deploy-file \
                            -s $MAVEN_SETTINGS_NEXUS_DEV \
                            -DgroupId='${GROUP_ID}' \
                            -DartifactId='${ARTIFACT_ID}' \
                            -Dversion='${VERSION_TOBE_RELEASED}' \
                            -DgeneratePom=true \
                            -Dpackaging=jar \
                            -DrepositoryId='${NEXUS_ID}' \
                            -Durl='${NEXUS_SERVER}' \
                            -Dfile=./target/${pom.artifactId}-${pom.version}-mule-application.jar'
                    }
                }
            }
            post {
                success {
                    echo "...Upload to Nexus is Succeeded for ${env.BUILD_VERSION}: ${currentBuild.currentResult}"
                }
                unsuccessful {
                    echo "...Upload to Nexus is Failed for ${env.BUILD_VERSION}: ${currentBuild.currentResult}"
                }
            }
        }

        stage('Release:Prepare') {
            when {
                expression { params.RELEASE_READY == 'Y' }
            }
            steps {
                script {
                    echo "Release:Prepare"
                    /*
                    def pom = readMavenPom file: 'pom.xml'
                    sh "git remote -v"
                    sh "git remote remove origin"
                    sh "git remote set-url origin https://gitlab.rhgcs.com/mulesoft-commons/rh-ati-development-template.git"
                    sh "git checkout -b origin/$BRANCH_NAME"
                    sh "git tag -a ${pom.version} -m 'new release'"
                    sh "git push --tags"
                    */
                    // sh "mvn --batch-mode release:update-versions -s $MVN_SET"
                    sh "mvn --batch-mode release:clean release:prepare -s $MVN_SET -DskipTests=true -Dmaven.deploy.skip=true"
                }
            }
            post {
                success {
                    echo "...Release:Prepare is Succeeded for ${env.BUILD_VERSION}: ${currentBuild.currentResult}"
                }
                unsuccessful {
                    echo "...Release:Prepare is Failed for ${env.BUILD_VERSION}: ${currentBuild.currentResult}"
                }
            }
        }



        stage('Deploy Artifact') {
            when {
                expression { params.DEPLOY_READY == 'Y' }
            }
            steps {
                script {
                    env.CLOUDHUB_WORKER_TYPE_ENV = "${params.'CLOUDHUB_WORKER_TYPE'}"
                    env.CLOUDHUB_WORKER_TYPE_TOKEN =CLOUDHUB_WORKER_TYPE_ENV.split("\\(")[0]
                    echo '$CLOUDHUB_WORKER_TYPE_TOKEN'

                    def pom = readMavenPom file: 'pom.xml'
                    env.GROUP_ID = pom.groupId
                    env.ARTIFACT_ID = pom.artifactId
                    env.VERSION = pom.version


                    if (params.RELEASE_VERSION == '' ) {
                        echo "RELEASE_VERSION is empty, deploying from build"
                        sh "mvn -U -V -e -B -DskipTests clean package install deploy -DmuleDeploy \
                            -s  $MVN_SET \
                            -DEnvironmentClientId='$CONNECTED_APP_CREDS_USR' \
                            -DEnvironmentClientSecret='$CONNECTED_APP_CREDS_PSW' \
                            -Dcloudhub.application.name=rh-ati-development-template \
                            -Dcloudhub.environment='${params.TARGET_CLOUDHUB_ENVIRONMENT}' \
                            -Dbusiness.group.name='${params.ANYPOINT_BUSINESS_GROUP}' \
                            -Dcloudhub.workers='${params.CLOUDHUB_WORKERS}' \
                            -Dcloudhub.worker.type='$CLOUDHUB_WORKER_TYPE_TOKEN' \
                            -Dcloudhub.region='${CLOUDHUB_REGION}' \
                            -Dskip.deploy.verify=true \
                            -Danypoint.platform.client.id='${params.ANYPOINT_PLATFORM_CLIENT_ID}' \
                            -Danypoint.platform.client.secret='${params.ANYPOINT_PLATFORM_CLIENT_SECRET}' \
                            -Denable.anypoint.monitoring='${params.ANYPOINT_MONITORING}' \
                            -Dapi.id='${params.API_ID}'"
                        echo "Artifact Deployed: ${currentBuild.currentResult}"
                    } else {
                        echo "RELEASE_VERSION is provided, deploying artifact from RH nexus"
                        //def artifactURL = NEXUS_SERVER + "/" + GROUP_ID + "/" +  ARTIFACT_ID + "" + "ARTIFACT_ID" + "-" +
                        //sh 'curl https://nexus.rhgcs.com/repository/rh-sfcc-maven-hosted/bb7d2e7e-a933-4a94-b345-6effe4a58055/rh-ati-development-template/1.0.1-SNAPSHOT/rh-ati-development-template-1.0.1-20221104.033125-1.jar -o ./target/mule-application.jar'

                        sh """ mvn -s $MVN_SET dependency:copy \
                            -Dartifact="${GROUP_ID}:${ARTIFACT_ID}:${params.RELEASE_VERSION}:jar:mule-application" \
                            -DrepositoryId='${NEXUS_ID}' \
                            -Durl='${NEXUS_SERVER}' """

                        sh "mvn --batch-mode -DskipTests deploy -DmuleDeploy \
                            -s  $MVN_SET \
                            -Dartifact.path=./target/dependency/'${ARTIFACT_ID}'-'${params.RELEASE_VERSION}'-mule-application.jar \
                            -DEnvironmentClientId='$CONNECTED_APP_CREDS_USR' \
                            -DEnvironmentClientSecret='$CONNECTED_APP_CREDS_PSW' \
                            -Dcloudhub.application.name=rh-ati-development-template \
                            -Dcloudhub.environment='${params.TARGET_CLOUDHUB_ENVIRONMENT}' \
                            -Dbusiness.group.name='${params.ANYPOINT_BUSINESS_GROUP}' \
                            -Dcloudhub.workers='${params.CLOUDHUB_WORKERS}' \
                            -Dcloudhub.worker.type='$CLOUDHUB_WORKER_TYPE_TOKEN' \
                            -Dcloudhub.region='${CLOUDHUB_REGION}' \
                            -Dskip.deploy.verify=true \
                            -Danypoint.platform.client.id='${params.ANYPOINT_PLATFORM_CLIENT_ID}' \
                            -Danypoint.platform.client.secret='${params.ANYPOINT_PLATFORM_CLIENT_SECRET}' \
                            -Denable.anypoint.monitoring='${params.ANYPOINT_MONITORING}' \
                            -Dapi.id='${params.API_ID}'"
                        echo "Artifact Deployed: ${currentBuild.currentResult}"
                    }
                }
            }
            post {
                success {
                    echo "...Deploy Artifact Succeeded for ${env.BUILD_VERSION}: ${currentBuild.currentResult}"
                }
                unsuccessful {
                    echo "...Deploy Artifact Failed for ${env.BUILD_VERSION}: ${currentBuild.currentResult}"
                }
            }
        }

        stage('Regression Suits') {
            when {
                expression { params.REGRESSION_SUITE_READY == 'Y' }
            }
            steps {
                echo "Queue creation validation and creation"
                sh 'newman run postman/ci-cd/regression-suits.postman-collection.json'
            }
            post {
                success {
                    echo "...Regression Suits is Succeeded for ${env.BUILD_VERSION}: ${currentBuild.currentResult}"
                }
                unsuccessful {
                    echo "...Regression Suits is Failed for ${env.BUILD_VERSION}: ${currentBuild.currentResult}"
                }
            }
        }
    }
    post {
        success {
            echo "All Good: ${env.BUILD_VERSION}: ${currentBuild.currentResult}"
        }
        failure {
            echo "Not So Good: ${env.BUILD_VERSION}:? ${currentBuild.currentResult}"
        }
        always {
            echo "Pipeline result: ${currentBuild.result}"
            echo "Pipeline currentResult: ${currentBuild.currentResult}"
        }
    }
}

def notifyUserMunit() {
     echo "send standard build failure email notification"
}