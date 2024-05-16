pipeline {
    agent any

    sh 'set +x'
    stages {
        stage('Precheck') {
            steps {
                echo '*** Preliminary steps ***'
                echo 'Checking tool versions does two things:'
                echo ' * Documents versions used for this run'
                echo ' * Shows that the tools are installed and work'
                sh 'echo Jenkins version: `jenkins --version`' 
            }
        }
        
        // did this in Jenkins
//        stage('Clone') {
//            steps {
//                git 'https://github.com/xenloops/password-vault'
//            }
//        }
     
        stage('Build') {
            steps {
                echo '*** Building the project ***'
                sh 'echo ant version: `ant -version`' 
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
                sh 'tar -zcf binaries.tar.gz build/classes/passvault'
                sh 'echo shasum version: `shasum --version`' 
                sh 'shasum -a 256 binaries.tar.gz > binaries.hash'
                sh 'rm binaries.tar.gz'
                echo 'Hash of binary files:'
                sh 'cat binaries.hash'
                echo 'Still need to compare with independent hash!'
                echo 'We\'ll leave that for a homework assignment. :)'
            }
        }
     
        stage('SBOM') {
            steps {
                echo '*** Generating SBOM ***'
                sh 'echo CycloneDX cdxgen version: `cdxgen --version`'
                sh 'cdxgen -o password-vault-bom.json'
            }
        }
        stage ('SCA') {
            steps {
                echo '*** Checking dependencies ***'
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
