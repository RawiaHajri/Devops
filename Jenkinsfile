def commit_id
pipeline {
    agent any
    stages {
        stage('preparation') {
            steps {
                checkout scm
                sh "git rev-parse --short HEAD > .git/commit-id"
                script {
                    commit_id = readFile('.git/commit-id').trim()
                }
            }
        }
	    
	 stage ('code quality') {
            steps {
                echo 'testing code quality'
                sh "mvn clean verify sonar:sonar \
                    -Dsonar.projectKey=position-similator \
                    -Dsonar.host.url=http://localhost:9000 \
                    -Dsonar.login=fa195595f77e1e28e39372864440d1f1fa82be0a"
                echo 'code quality test complete'
            }
        }
	    
	    
        stage ('build') {
            steps {
                echo 'building maven workload'
                sh "mvn clean install"
                echo 'build complete'
            }
        }

        stage ("image build") {
            steps {
                echo 'building docker image'
                sh "docker build -t rawiahajri/position-simulator:${commit_id} ."
		sh "docker push rawiahajri/position-simulator:${commit_id}"
                echo 'docker image built'
            }
        }
        
        	
        stage('deploy') {
	          steps {
	             sh "sed -i -r 's|richardchesterwood/k8s-fleetman-position-simulator:release2|rawiahajri/position-simulator:${commit_id}|' workloads.yaml"
	             sh "kubectl apply -f workloads.yaml"
            }
        }
    }
}
