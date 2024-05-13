pipeline {
    agent any

    environment {
        // Grab service account DB login from Jenkins secret storage
        DB_USERNAME = credentials('test_service_account').username
        DB_PASSWORD = credentials('test_service_account').password
    }

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
                sh 'echo "Username: $DB_USERNAME"'
                sh 'echo "Password: $DB_PASSWORD"'
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
                    -o build/classes/passvault 
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
