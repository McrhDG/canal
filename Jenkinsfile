pipeline {
    agent any
    stages {
        stage("编译打包") {
                   steps {
                       script {
                            sh "curl -i -X POST -H \"'Content-type':'application/json'\" -d \'{\"buildid\":\"${env.BUILD_ID}\",\"jenkinsjobname\":\"${env.JOB_NAME}\",\"buildurl\":\"${env.BUILD_URL}\",\"branchname\":\"${env.GIT_BRANCH}\",\"images\":\"\",\"status\":\"构建中\",\"steps\":\"start\"}\' http://10.200.0.128:80/project/jenkins/buildInfo"
                            sh './docker/build.sh admin'
                            sh './docker/build.sh server'
                       }
                   }
               }

        stage("上传项目包") {
            steps {
                script {
                    def projectBranch = env.GIT_BRANCH.split("/")[1]
                    def projectName = env.JOB_NAME
                    def pom = readMavenPom file: 'pom.xml'
                    def version = pom.version
                    def contentRegex = "/.{0,10}/"
                    def commit = "${env.GIT_COMMIT}"
                    def commitSha1_10 = sh label: "", returnStdout: true, script: "expr substr '${commit}' 1 10"
                    commitSha1_10=commitSha1_10.replaceAll("\n","")
                    def timeStr = new Date().format('yyyyMMddHHmm')
                    def ImageName1="wenwo/canal-admin:v1.1.5"
                    def ImageName2="wenwo/canal-server:v1.1.5"

                    def aliyunImageName1="registry.cn-beijing.aliyuncs.com/wenwo/canal-admin:v1.1.5"
                    def aliyunImageName2="registry.cn-beijing.aliyuncs.com/wenwo/canal-server:v1.1.5"

                    
                    sh "echo '' > Dockerfile"
                    sh "echo 'FROM registry.cn-beijing.aliyuncs.com/awyl/java:v1' >> Dockerfile"
                    sh "echo 'VOLUME /tmp' >> Dockerfile"
                    sh "echo 'COPY ${jarName} app.jar' >> Dockerfile"
                    sh "echo 'ENV JAVA_OPTS=\"\"' >> Dockerfile"
                    sh "echo 'ENV Active=\"dev\"' >> Dockerfile"
                    sh "echo 'ENV IP=\"\"' >> Dockerfile"
                    sh "echo 'ENV ZONE=\"http://192.168.1.240:7001/eureka\"' >> Dockerfile"
                    sh "echo 'RUN [\"/bin/mkdir\", \"-p\", \"/data/logs/\"]' >> Dockerfile"
                    sh "echo 'ENTRYPOINT [\"sh\",\"-c\",\"java \$JAVA_OPTS -Dspring.profiles.active=\$Active -jar app.jar \"]' >> Dockerfile"
                    sh "echo 'EXPOSE ${PORT}' >> Dockerfile"
                    sh "cat Dockerfile"
                    sh "mv Dockerfile target/"
                    sh "cd target/ && docker build -t ${ImageName} ."
                    sh "docker push ${ImageName}"
                    sh "docker rmi ${ImageName}"
                    sh "curl -i -X POST -H \"'Content-type':'application/json'\" -d \'{\"buildid\":\"${env.BUILD_ID}\",\"jenkinsjobname\":\"${env.JOB_NAME}\",\"buildurl\":\"${env.BUILD_URL}\",\"branchname\":\"${env.GIT_BRANCH}\",\"images\":\"${ImageName}\",\"status\":\"构建成功\",\"steps\":\"end\"}\' http://10.200.0.128:80/project/jenkins/buildInfo"

                }
            }
        }

        stage("清理空间") {
            steps {
                sh "ls -al"
                deleteDir()
                sh "ls -al"
            }
        }
    }
    post {
        failure {
            sh "curl -i -X POST -H \"'Content-type':'application/json'\" -d \'{\"buildid\":\"${env.BUILD_ID}\",\"jenkinsjobname\":\"${env.JOB_NAME}\",\"buildurl\":\"${env.BUILD_URL}\",\"branchname\":\"${env.GIT_BRANCH}\",\"images\":\"\",\"status\":\"构建失败\",\"steps\":\"end\"}\' http://10.200.0.128:80/project/jenkins/buildInfo"
        }
    }
}
