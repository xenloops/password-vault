pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo '***************************'
                echo '*** Building the project...'
                echo '***************************'
              sh 'ant compile jar'
            }
        }
        stage ('SCA') {
            steps {
                echo '***************************'
                echo '*** Checking dependencies...'
                echo '***************************'
                dependencyCheck additionalArguments: ''' 
                    -o "./" 
                    -s "./"
                    -f "ALL" 
                    --prettyPrint''', odcInstallation: 'Dependency-Check'
                dependencyCheckPublisher pattern: 'dependency-check-report.xml'
        stage('SAST') {
            steps {
                echo '************************'
                echo '*** Running SonarQube...'
                echo '************************'
            }
        }
    }
}
