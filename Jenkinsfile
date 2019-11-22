pipeline {
    agent none

    parameters {
        string(defaultValue: "https://github.com/Florin-AndreiMihai/cubrid.git", description: 'Whats the github URL?', name: 'GitHubURL')
        string(defaultValue: "develop", description: 'Whats the Branch Name?', name: 'BranchName')
        choice(name: 'Build', choices: ['All', 'Debug', 'Release'], description: 'Build specific job')
    }

    triggers {
        pollSCM('H 1 * * *')
    }

    environment {
        OUTPUT_DIR = 'packages'
        TEST_REPORT = 'reports'
    }

    stages {
        stage('Build') {
            parallel {
                stage('Release') {
                    when {
                        expression {
                            params.Build == 'Release' || params.Build == 'All'
                        }
                    }
                    agent {
                        docker {
                            image 'florinmihai/dockerci:develop'
                            label 'linux'
                            alwaysPull true
                            args '--init'
                            registryUrl 'https://index.docker.io/v1/'
                            registryCredentialsId '0b8aefb7-bb51-4cfa-8d02-0edbb5ba980f'
                        }
                    }
                    environment {
                        MAKEFLAGS = '-j'
                    }
                    steps {
                        script {
                            currentBuild.displayName = sh(returnStdout: true, script: './build.sh -v').trim()
                        }

                        echo 'Get repository...'
                        deleteDir()
                        git branch: "${params.BranchName}", url: "${params.GitHubURL}"
                        
                        echo 'Get Cubrid Manager...'
                        dir('cubridmanager') {
                            git branch: 'develop', url: 'https://github.com/CUBRID/cubrid-manager-server.git'
                        }

                        echo 'Building...'
                        sh "scl enable devtoolset-8 -- /entrypoint.sh build"

                        echo 'Packing...'
                        sh "scl enable devtoolset-8 -- /entrypoint.sh dist -o ${OUTPUT_DIR}"

                        echo 'Stashing Build...'
                        stash includes: "${OUTPUT_DIR}/*", name: "build_release"

                    }

                    post {
                        success {
                            archiveArtifacts "${OUTPUT_DIR}/*"
                        }
                    }
                }
                stage('Debug') {
                    when {
                        expression {
                            params.Build == 'Debug' || params.Build == 'All'
                        }
                    }
                    agent {
                        docker {
                            image 'florinmihai/dockerci:develop'
                            label 'linux'
                            alwaysPull true
                            args '--init'
                            registryUrl 'https://index.docker.io/v1/'
                            registryCredentialsId '0b8aefb7-bb51-4cfa-8d02-0edbb5ba980f'
                        }
                    }
                    environment {
                        MAKEFLAGS = '-j'
                    }
                    steps {

                        script {
                            currentBuild.displayName = sh(returnStdout: true, script: './build.sh -v').trim()
                        }

                        echo 'Get repository...'
                        deleteDir()
                        git branch: "${params.BranchName}", url: "${params.GitHubURL}"

                        echo 'Get Cubrid Manager...'
                        dir('cubridmanager') {
                            git branch: 'develop', url: 'https://github.com/CUBRID/cubrid-manager-server.git'
                        }

                        echo 'Building...'
                        sh "scl enable devtoolset-8 -- /entrypoint.sh build -m debug"

                        echo 'Packing...'
                        sh "scl enable devtoolset-8 -- /entrypoint.sh dist -m debug -o ${OUTPUT_DIR}"

                        echo 'Stashing Build...'
                        stash includes: "${OUTPUT_DIR}/*", name: "build_debug"

                    }

                    post {
                        success {
                            archiveArtifacts "${OUTPUT_DIR}/*"
                        }
                    }
                }
            }
        }

        stage('Prep Test Data') {
            agent {
                docker {
                    image 'florinmihai/dockerci:develop'
                    label 'linux'
                    args '--init'
                    registryUrl 'https://index.docker.io/v1/'
                    registryCredentialsId '0b8aefb7-bb51-4cfa-8d02-0edbb5ba980f'
                }
            }
            steps {

                echo 'Pulling tastcases...'
                dir('cubrid-testcases') {
                    git branch: 'develop', url: 'https://github.com/CUBRID/cubrid-testcases.git'
                }

                echo 'Archiving medium tests...'
                sh 'tar -czf cubrid-testcases.tar.gz cubrid-testcases/medium cubrid-testcases/sql/_02_user_authorization cubrid-testcases/sql/_03_object_oriented cubrid-testcases/sql/_16_index_enhancement cubrid-testcases/sql/_17_sql_extension2 cubrid-testcases/sql/_18_index_enhancement_qa cubrid-testcases/sql/_19_apricot cubrid-testcases/sql/config cubrid-testcases/LICENSE.md cubrid-testcases/.git'

                echo 'Stashing medium tests...'
                stash includes: 'cubrid-testcases.tar.gz', name: "cubridTC_medium"

                echo 'Archiving sql 1 tests...'
                sh 'tar -czf cubrid-testcases1.tar.gz cubrid-testcases/sql/_01_object cubrid-testcases/sql/_04_operator_function cubrid-testcases/sql/_06_manipulation cubrid-testcases/sql/_07_misc cubrid-testcases/sql/_08_javasp cubrid-testcases/LICENSE.md cubrid-testcases/sql/config cubrid-testcases/.git'

                echo 'Stashing sql 1 tests...'
                stash includes: 'cubrid-testcases1.tar.gz', name: "cubridTC_sql1"

                echo 'Archiving sql 2 tests...'
                sh 'tar -czf cubrid-testcases2.tar.gz cubrid-testcases/sql/_09_64bit cubrid-testcases/sql/_10_connect_by cubrid-testcases/sql/_11_codecoverage cubrid-testcases/sql/_12_mysql_compatibility cubrid-testcases/sql/_13_issues cubrid-testcases/sql/_14_mysql_compatibility_2 cubrid-testcases/sql/_15_fbo cubrid-testcases/LICENSE.md cubrid-testcases/sql/config cubrid-testcases/.git'

                echo 'Stashing sql 2 tests...'
                stash includes: 'cubrid-testcases2.tar.gz', name: "cubridTC_sql2"

                echo 'Archiving sql 3 tests...'
                sh 'tar -czf cubrid-testcases3.tar.gz cubrid-testcases/sql/_22_news_service_mysql_compatibility cubrid-testcases/sql/_23_apricot_qa cubrid-testcases/sql/_24_aprium_qa cubrid-testcases/LICENSE.md cubrid-testcases/sql/config cubrid-testcases/.git'

                echo 'Stashing sql 3 tests...'
                stash includes: 'cubrid-testcases3.tar.gz', name: "cubridTC_sql3"

                echo 'Archiving sql 4 tests...'
                sh 'tar -czf cubrid-testcases4.tar.gz cubrid-testcases/sql/_25_features_844/issue_9725_add_not_null_column cubrid-testcases/sql/_26_features_920 cubrid-testcases/sql/_27_banana_qa cubrid-testcases/sql/_28_features_930 cubrid-testcases/sql/_29_CTE_recursive cubrid-testcases/sql/_30_banana_pie_qa cubrid-testcases/sql/_31_cherry cubrid-testcases/LICENSE.md cubrid-testcases/sql/config cubrid-testcases/.git'

                echo 'Stashing sql 4 tests...'
                stash includes: 'cubrid-testcases4.tar.gz', name: "cubridTC_sql4"
            }
        }

        stage('Testing') {
            parallel {
                stage('Linux Release medium') {
                    when {
                        expression {
                            params.Build == 'Release' || params.Build == 'All'
                        }
                    }
                    agent {
                        docker {
                            image 'florinmihai/dockerci:develop'
                            label 'linux'
                            args '--init'
                            registryUrl 'https://index.docker.io/v1/'
                            registryCredentialsId '0b8aefb7-bb51-4cfa-8d02-0edbb5ba980f'
                        }
                    }
                    steps {

                        echo 'Unstashing build...'
                        unstash 'build_release'

                        echo 'Unzipping...'
                        sh 'tar -xzf /var/jenkins_home/workspace/cubrid_develop/packages/CUBRID-10.2.0.*-Linux.x86_64.tar.gz -C /home'

                        echo 'Unstashing curbidTC...'
                        unstash 'cubridTC_medium'

                        echo 'Unzipping...'
                        sh 'tar -xzf /var/jenkins_home/workspace/cubrid_develop/cubrid-testcases.tar.gz -C /home'

                        echo 'Testing...'
                        sh '/entrypoint.sh test || echo "$? failed"'

                    }

                    post {
                        always {
                            archiveArtifacts "${OUTPUT_DIR}/*"
                            junit "${TEST_REPORT}/*.xml"
                        }
                    }
                }

                stage('Linux Release sql1') {
                    when {
                        expression {
                            params.Build == 'Release' || params.Build == 'All'
                        }
                    }
                    agent {
                        docker {
                            image 'florinmihai/dockerci:develop'
                            label 'linux'
                            args '--init'
                            registryUrl 'https://index.docker.io/v1/'
                            registryCredentialsId '0b8aefb7-bb51-4cfa-8d02-0edbb5ba980f'
                        }
                    }
                    steps {

                        echo 'Unstashing build...'
                        unstash 'build_release'

                        echo 'Unzipping...'
                        sh 'tar -xzf /var/jenkins_home/workspace/cubrid_develop@2/packages/CUBRID-10.2.0.*-Linux.x86_64.tar.gz -C /home'

                        echo 'Unstashing curbidTC...'
                        unstash 'cubridTC_sql1'

                        echo 'Unzipping...'
                        sh 'tar -xzf /var/jenkins_home/workspace/cubrid_develop@2/cubrid-testcases1.tar.gz -C /home'

                        echo 'Testing...'
                        sh '/entrypoint.sh test || echo "$? failed"'

                    }

                    post {
                        always {
                            junit "${TEST_REPORT}/*.xml"
                        }
                    }
                }

                stage('Linux Release sql2') {
                    when {
                        expression {
                            params.Build == 'Release' || params.Build == 'All'
                        }
                    }
                    agent {
                        docker {
                            image 'florinmihai/dockerci:develop'
                            label 'linux'
                            args '--init'
                            registryUrl 'https://index.docker.io/v1/'
                            registryCredentialsId '0b8aefb7-bb51-4cfa-8d02-0edbb5ba980f'
                        }
                    }
                    steps {

                        echo 'Unstashing build...'
                        unstash 'build_release'

                        echo 'Unzipping...'
                        sh 'tar -xzf /var/jenkins_home/workspace/cubrid_develop@3/packages/CUBRID-10.2.0.*-Linux.x86_64.tar.gz -C /home'

                        echo 'Unstashing curbidTC...'
                        unstash 'cubridTC_sql2'

                        echo 'Unzipping...'
                        sh 'tar -xzf /var/jenkins_home/workspace/cubrid_develop@3/cubrid-testcases2.tar.gz -C /home'

                        echo 'Testing...'
                        sh '/entrypoint.sh test || echo "$? failed"'

                    }

                    post {
                        always {
                            junit "${TEST_REPORT}/*.xml"
                        }
                    }
                }

                stage('Linux Release sql3') {
                    when {
                        expression {
                            params.Build == 'Release' || params.Build == 'All'
                        }
                    }
                    agent {
                        docker {
                            image 'florinmihai/dockerci:develop'
                            label 'linux'
                            args '--init'
                            registryUrl 'https://index.docker.io/v1/'
                            registryCredentialsId '0b8aefb7-bb51-4cfa-8d02-0edbb5ba980f'
                        }
                    }
                    steps {

                        echo 'Unstashing build...'
                        unstash 'build_release'

                        echo 'Unzipping...'
                        sh 'tar -xzf /var/jenkins_home/workspace/cubrid_develop@4/packages/CUBRID-10.2.0.*-Linux.x86_64.tar.gz -C /home'

                        echo 'Unstashing curbidTC...'
                        unstash 'cubridTC_sql3'

                        echo 'Unzipping...'
                        sh 'tar -xzf /var/jenkins_home/workspace/cubrid_develop@4/cubrid-testcases3.tar.gz -C /home'

                        echo 'Testing...'
                        sh '/entrypoint.sh test || echo "$? failed"'

                    }

                    post {
                        always {
                            junit "${TEST_REPORT}/*.xml"
                        }
                    }
                }

                stage('Linux Release sql4') {
                    when {
                        expression {
                            params.Build == 'Release' || params.Build == 'All'
                        }
                    }
                    agent {
                        docker {
                            image 'florinmihai/dockerci:develop'
                            label 'linux'
                            args '--init'
                            registryUrl 'https://index.docker.io/v1/'
                            registryCredentialsId '0b8aefb7-bb51-4cfa-8d02-0edbb5ba980f'
                        }
                    }
                    steps {

                        echo 'Unstashing build...'
                        unstash 'build_release'

                        echo 'Unzipping...'
                        sh 'tar -xzf /var/jenkins_home/workspace/cubrid_develop@5/packages/CUBRID-10.2.0.*-Linux.x86_64.tar.gz -C /home'

                        echo 'Unstashing curbidTC...'
                        unstash 'cubridTC_sql4'

                        echo 'Unzipping...'
                        sh 'tar -xzf /var/jenkins_home/workspace/cubrid_develop@5/cubrid-testcases4.tar.gz -C /home'

                        echo 'Testing...'
                        sh '/entrypoint.sh test || echo "$? failed"'

                    }

                    post {
                        always {
                            junit "${TEST_REPORT}/*.xml"
                        }
                    }
                }

                stage('Linux Debug medium') {
                    when {
                        expression {
                            params.Build == 'Debug' || params.Build == 'All'
                        }
                    }
                    agent {
                        docker {
                            image 'florinmihai/dockerci:develop'
                            label 'linux'
                            args '--init'
                            registryUrl 'https://index.docker.io/v1/'
                            registryCredentialsId '0b8aefb7-bb51-4cfa-8d02-0edbb5ba980f'
                        }
                    }
                    steps {

                        echo 'Unstashing...'
                        unstash 'build_debug'

                        echo 'Unzipping...'
                        sh 'tar -xzf /var/jenkins_home/workspace/cubrid_develop@6/packages/CUBRID-10.2.0.*-Linux.x86_64-debug.tar.gz -C /home'

                        echo 'Unstashing curbidTC...'
                        unstash 'cubridTC_medium'

                        echo 'Unzipping...'
                        sh 'tar -xzf /var/jenkins_home/workspace/cubrid_develop@6/cubrid-testcases.tar.gz -C /home'

                        echo 'Testing...'
                        sh '/entrypoint.sh test || echo "$? failed"'
                    }

                    post {
                        always {
                            junit "${TEST_REPORT}/*.xml"
                        }
                    }
                }

                stage('Linux Debug sql1') {
                    when {
                        expression {
                            params.Build == 'Debug' || params.Build == 'All'
                        }
                    }
                    agent {
                        docker {
                            image 'florinmihai/dockerci:develop'
                            label 'linux'
                            args '--init'
                            registryUrl 'https://index.docker.io/v1/'
                            registryCredentialsId '0b8aefb7-bb51-4cfa-8d02-0edbb5ba980f'
                        }
                    }
                    steps {

                        echo 'Unstashing...'
                        unstash 'build_debug'

                        echo 'Unzipping...'
                        sh 'tar -xzf /var/jenkins_home/workspace/cubrid_develop@7/packages/CUBRID-10.2.0.*-Linux.x86_64-debug.tar.gz -C /home'

                        echo 'Unstashing curbidTC...'
                        unstash 'cubridTC_sql1'

                        echo 'Unzipping...'
                        sh 'tar -xzf /var/jenkins_home/workspace/cubrid_develop@7/cubrid-testcases1.tar.gz -C /home'

                        echo 'Testing...'
                        sh '/entrypoint.sh test || echo "$? failed"'
                    }

                    post {
                        always {
                            junit "${TEST_REPORT}/*.xml"
                        }
                    }
                }

                stage('Linux Debug sql2') {
                    when {
                        expression {
                            params.Build == 'Debug' || params.Build == 'All'
                        }
                    }
                    agent {
                        docker {
                            image 'florinmihai/dockerci:develop'
                            label 'linux'
                            args '--init'
                            registryUrl 'https://index.docker.io/v1/'
                            registryCredentialsId '0b8aefb7-bb51-4cfa-8d02-0edbb5ba980f'
                        }
                    }
                    steps {

                        echo 'Unstashing...'
                        unstash 'build_debug'

                        echo 'Unzipping...'
                        sh 'tar -xzf /var/jenkins_home/workspace/cubrid_develop@8/packages/CUBRID-10.2.0.*-Linux.x86_64-debug.tar.gz -C /home'

                        echo 'Unstashing curbidTC...'
                        unstash 'cubridTC_sql2'

                        echo 'Unzipping...'
                        sh 'tar -xzf /var/jenkins_home/workspace/cubrid_develop@8/cubrid-testcases2.tar.gz -C /home'

                        echo 'Testing...'
                        sh '/entrypoint.sh test || echo "$? failed"'
                    }

                    post {
                        always {
                            junit "${TEST_REPORT}/*.xml"
                        }
                    }
                }

                stage('Linux Debug sql3') {
                    when {
                        expression {
                            params.Build == 'Debug' || params.Build == 'All'
                        }
                    }
                    agent {
                        docker {
                            image 'florinmihai/dockerci:develop'
                            label 'linux'
                            args '--init'
                            registryUrl 'https://index.docker.io/v1/'
                            registryCredentialsId '0b8aefb7-bb51-4cfa-8d02-0edbb5ba980f'
                        }
                    }
                    steps {

                        echo 'Unstashing...'
                        unstash 'build_debug'

                        echo 'Unzipping...'
                        sh 'tar -xzf /var/jenkins_home/workspace/cubrid_develop@9/packages/CUBRID-10.2.0.*-Linux.x86_64-debug.tar.gz -C /home'

                        echo 'Unstashing curbidTC...'
                        unstash 'cubridTC_sql3'

                        echo 'Unzipping...'
                        sh 'tar -xzf /var/jenkins_home/workspace/cubrid_develop@9/cubrid-testcases3.tar.gz -C /home'

                        echo 'Testing...'
                        sh '/entrypoint.sh test || echo "$? failed"'
                    }

                    post {
                        always {
                            junit "${TEST_REPORT}/*.xml"
                        }
                    }
                }

                stage('Linux Debug sql4') {
                    when {
                        expression {
                            params.Build == 'Debug' || params.Build == 'All'
                        }
                    }
                    agent {
                        docker {
                            image 'florinmihai/dockerci:develop'
                            label 'linux'
                            args '--init'
                            registryUrl 'https://index.docker.io/v1/'
                            registryCredentialsId '0b8aefb7-bb51-4cfa-8d02-0edbb5ba980f'
                        }
                    }
                    steps {

                        echo 'Unstashing...'
                        unstash 'build_debug'

                        echo 'Unzipping...'
                        sh 'tar -xzf /var/jenkins_home/workspace/cubrid_develop@10/packages/CUBRID-10.2.0.*-Linux.x86_64-debug.tar.gz -C /home'

                        echo 'Unstashing curbidTC...'
                        unstash 'cubridTC_sql4'

                        echo 'Unzipping...'
                        sh 'tar -xzf /var/jenkins_home/workspace/cubrid_develop@10/cubrid-testcases4.tar.gz -C /home'

                        echo 'Testing...'
                        sh '/entrypoint.sh test || echo "$? failed"'
                    }

                    post {
                        always {
                            junit "${TEST_REPORT}/*.xml"
                        }
                    }
                }
            }
        }
    }

    post {
        always {
            //build job: "${DEPLOY_JOB}", parameters: [string(name: 'PROJECT_NAME', value: "${JOB_NAME}")],
            //propagate: false
            emailext replyTo: '$DEFAULT_REPLYTO', to: '$DEFAULT_RECIPIENTS',
                subject: '$DEFAULT_SUBJECT', body: '''${JELLY_SCRIPT,template="html"}'''
        }
    }
}
