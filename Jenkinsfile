def buildImage(cluster_name, env_name) {
	openshift.withCluster( cluster_name ) {
		openshift.withProject( env_name ) {
			echo "Building Image Stream from $APP_NAME"
			
			def result = openshift.selector("bc", "$APP_NAME").startBuild("--follow")
			echo "Build result: ${result.status}"
			
			def dc = openshift.selector('dc', "$APP_NAME")
			dc.rollout().status()
		}
	}
}

def tagImage(cluster_name, env_name, action) {
	openshift.withCluster( cluster_name ) {
		openshift.withProject( env_name ) {
			if (action == 'testing') {
				echo "Tag image from development to testing."
				openshift.tag("development/$APP_NAME:latest", "testing/$APP_NAME:test")
				def dc = openshift.selector('dc', "$APP_NAME")
				dc.rollout().status()
			}
			if (action == 'production') {
				echo "Tag image from testing to production."
				openshift.tag("testing/$APP_NAME:test", "production/$APP_NAME:prod")
				def dc = openshift.selector('dc', "$APP_NAME")
				dc.rollout().status()
			}
		}
	}
}

pipeline{
	agent any
	environment{
		def CLUSTER_NAME = "openshift-4.3" 
		def ENV_TEST = "sandbox"
		def ENV_PROD = "idc"
	}
	stages{ 
		stage ("Build") {
			steps {
				script {
				    buildImage(CLUSTER_NAME, ENV_TEST)
				}
			}
		}
		stage('approval (sandbox)') {
			steps{
				input 'Approve for sandbox?'
			}
		}
		stage ("Deploy Sandbox") {
			steps {
				script {
					tagImage(CLUSTER_NAME, ENV_TEST, 'testing')
				}
			}
		}
		stage('approval (production)') {
			steps{
				input 'Approve for production?'
			}
		}
		stage ("Deploy IDC") {
			steps {
				script {
					tagImage(CLUSTER_NAME, ENV_PROD, 'production')
				}
			}
		}
	}
}
