#!groovy
/** To-Do
 * 1. utilize params.region to mapping values file
   2. utilize params.release_name to name release for installation
   3. make a secret with kubernetes config file.
*/
def label = "install-mariadb-zabbix"

try{
	if (params.helm == "---"){
		error("You must select helm command 'install' or 'reinstall'")
	}else if(params.region == "---"){
		error("You must select region")
	}else if(params.release_name == "" ){
		error("You must input release_name")
	}else if(params.namespace == "" ){
		error("You must input namespace")
	}else{
		echo "your request: $params.helm"
		echo "your request: $params.region"
		echo "your request: $params.release_name" 
		echo "your request: $params.namespace" 

		podTemplate(
				label: label,
				containers: [
					//dtzar/helm-kubectl:3.0.1 == helm v3.0.1, kubectl v1.16.3, alpine 3.10
					containerTemplate(name: "helm", image: "dtzar/helm-kubectl:3.0.1", ttyEnabled: true, command: "cat")
				],
				//volume mount
				volumes: [
					secretVolume(secretName: 'kube-config', mountPath: '/root/.kube')
				]
		)
		{
			node(label) {
				try{
					checkout scm
					echo "Test.............."

					def helmRepoList
					def props = readProperties file: "./pipeline.properties"
					def helmRepositoryUrl = props["helmRepositoryUrl"]
					def helmRepository = props["helmRepository"]
					def helmChartName = props["helmChartName"]
					def releaseName = params.release_name
					def valuesDirectory = props["valuesDirectory"]
					def valuesFileName = "values-"+params.region+".yaml"
					def namespace = params.namespace
					def timeout = props["timeout"]
					if(params.timeout){
						timeout = params.timeout
					}

					echo "helmRepository:${helmRepository}"
					echo "helmChartName:${helmChartName}"
					echo "releaseName:${releaseName}"
					echo "valuesDirectory:${valuesDirectory}"
					echo "valuesFileName:${valuesFileName}"
					echo "namespace:${namespace}"
					echo "timeout:${timeout}"

					stage("Check Helm repository"){
						container("helm"){
							try{
								sh "helm repo add ${helmRepository} ${helmRepositoryUrl}"
								String out = sh script: "helm repo list -o json", returnStdout: true
								echo "helmRepoList : ${out}"
								helmRepoList = readJSON text: out
								echo "helmRepoList : ${helmRepoList[0].name}"
							}catch(e){
								error("Check Helm repository error : " + e.getMessage())
							}
						}
					}
						
					if(helmRepository == "${helmRepoList[0].name}"){
						if(params.helm == "reinstall"){
							stage( "Clean Up Existing Deployments" ) {
								container("helm") {
									boolean isExist = false
									//====== 이미 설치된 chart 인지 검사 =============
									String out = sh script: "helm ls -q --namespace ${namespace}", returnStdout: true
									if(out.contains("${releaseName}")) isExist = true
									//===========================
									if(isExist) {
										sh "helm delete ${releaseName} --namespace ${namespace}"
										sh "kubectl delete pvc data-${releaseName}-db-0 --namespace ${namespace}"
										sh "sleep 10"
									}else{
										echo "No ${releaseName} installed"
									}
								}
							}
						}

						if(params.helm == "upgrade"){
							stage("Upgrade Helm chart"){
								container("helm"){
									sh "helm upgrade ${releaseName} ${helmRepository}/${helmChartName} -f ${valuesDirectory}/${valuesFileName} -n ${namespace} --timeout ${timeout} --debug"	
								}
							}	
						}
						else{
							stage("Install Helm chart"){
								container("helm"){
									sh "helm install ${releaseName} ${helmRepository}/${helmChartName} -f ${valuesDirectory}/${valuesFileName} -n ${namespace} --timeout ${timeout} --debug"	
								}
							}	
						}
					}else{
						echo "There is no ${helmRepository} existed. Check the Helm repository name"
					}
				}catch (e){
					echo e.getMessage()
					currentBuild.result = "FAILED"
				}
			}//node(label)
		}
	}
}catch(e){
	echo e.getMessage()
	currentBuild.result = "FAILED"
}
