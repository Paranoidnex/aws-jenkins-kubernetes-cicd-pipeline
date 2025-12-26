pipeline{

agent any

environment{
    docker_image = "paranoidnex/python-app"
    tag = """${BUILD_NUMBER}"""
}
stages{
    
    stage("github checkout"){
        steps{
            git branch: 'main', credentialsId: 'github-cred', url: 'https://github.com/Paranoidnex/python-app.git'
        }
    }
    
    stage("docker image build"){
        steps{
            sh "docker build -t ${docker_image}:${tag} ."
        }
    }
    
    stage("docker push"){
        steps{
            withDockerRegistry(credentialsId: 'dockerhub-cred', url: 'https://index.docker.io/v1/') {
sh "docker push ${docker_image}:${tag}"
} } }

    stage("k8s yaml creation"){
        steps{
            fileOperations([fileCreateOperation(fileContent: """apiVersion: apps/v1
kind: Deployment metadata: name: python-app-deploy labels: app: python-app spec: replicas: 1 selector: matchLabels: app: python-app template: metadata: labels: app: python-app spec: containers: - name: python-app image: ${docker_image}:${tag} ports: - containerPort: 80
apiVersion: v1 kind: Service metadata: name: python-app-service spec: selector: app: python-app ports: - protocol: TCP port: 80 type: NodePort
""", fileName: 'deploy.yml')]) } }

    stage("k8s deplyment"){
        steps{
            script{
                kubernetesDeploy(configs:"deploy.yml",kubeconfigId:"kubeconfig")
            }
        }
    }
}
}
