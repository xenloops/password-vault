 pipeline {
    agent any

    //environment {
        // Grab service account DB login from Jenkins secret storage
        //USERNAME = credentials('test_service_account').username
        //PASSWORD = credentials('test_service_account').password
    //}

    stages {
        // did this in Jenkins
//        stage('Clone') {
//            steps {
//                git 'https://github.com/xenloops/password-vault'
//            }
//        }
     
        stage('Build') {
            steps {
                echo '*** Building the project...'
                sh 'ant compile jar'  //works
                //sh 'ant clean compile'
                // Super secret service account creds:
                echo '*** These credentials stored securely in Jenkins ***'
                withCredentials(
                    [usernamePassword(
                        credentialsId: 'test_service_account', 
                        usernameVariable: 'USERNAME', 
                        passwordVariable: 'PASSWORD')
                    ]) 
                {
                    sh 'echo "Username: $USERNAME"'
                    sh 'echo "Password: $PASSWORD"'
                }
                echo '*** Generating hash ***'
                sh 'ls -l'
                sh 'cd ..'
                echo 'One dir up'
                sh 'ls -l'
            }
        }
     
        stage('SBOM') {
            steps {
                echo '*** Generating SBOM ***'
                echo '(At least it will when this is working)'
                //sh 'cyclonedx generate-bom --output password-vault-SBOM.xml'
                //sh 'cdxgen bom --build-dir build --output-file password-vault-SBOM.xml'
                //sh 'mvn org.cyclonedx:cyclonedx-maven-plugin:makeAggregateBom'
                //sh 'pwd'
                //sh 'ls -l'
//                script {
                    //def cycloneDxHome = tool 'CycloneDX'
                    //sh "${cycloneDxHome}/cyclonedx-cli generate-bom --output bom.xml"
//                }
            }
        }
        stage ('SCA') {
            steps {
                echo '*** Checking dependencies...'
                dependencyCheck additionalArguments: ''' 
                    -o .
                    -s src/passvault 
                    -f ALL 
                    --prettyPrint''', odcInstallation: 'SCA: Dependency-Check'
                dependencyCheckPublisher pattern: 'dependency-check-report.xml'
            }
        }
        stage('SAST') {
            environment {
                SCANNER_HOME = tool 'SonarQube Scanner'
                PROJECT_KEY = 'password-vault'
            }
            steps {
                echo '*** Scanning the code...'
                // test command here
                withSonarQubeEnv('SonarQube server') {
                    sh '''$SCANNER_HOME/bin/sonar-scanner \
                    -Dsonar.projectKey=$PROJECT_KEY \
                    -Dsonar.projectName='Project analyzed by SonarQube' \
                    -Dsonar.projectVersion=1.0 \
                    -Dsonar.sources=src \
                    -Dsonar.language=java \
                    -Dsonar.sourceEncoding=UTF-8 \
                    -Dsonar.java.binaries=build/classes/passvault \
                    -Dsonar.java.libraries=dist/lib
                    '''
                }
            }
        }
    }
}
