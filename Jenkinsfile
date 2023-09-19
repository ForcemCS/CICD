pipeline{
    agent {
        kubernetes {
            inheritFrom 'maven-and-kubectl'
        }
    }
    
    parameters{
         booleanParam(name: 'imagePush', defaultValue: 'true', description: 'Push image  to Harbor')
     }
    /**
     *tools{
     *    maven 'maven-3.9.2'
     *}
     */
    /**
     *在进行测试的时候,需要允许本都gitlab触发(管理员账账号: settig--Network--Outbound requests(Allow requests to the local network from webhooks and integrations))
     *同时需要在gitlab 对应的项目之上设置webhook,来对接到jenkins的具体的job
     *example : URL http://jenkins.wukui.com:8080/project/pipeline-job-01
     *Secret token  wukui123456
     *若要实现addVoteOnMergeRequest: true 则需要生成一个gitlab api的访问令牌glpat-h1MXz5TarxPzoegM-Qx7，然后在jenkins上配置一个类型为secret text类型的凭据 \
     * 然后在系统配置中进行相关设置
     */
    triggers{      //具体配置参见插件官方文档
        gitlab(
            triggerOnPush: true,
            triggerOnMergeRequest: true,
            branchFilterType: 'All',
            addVoteOnMergeRequest: true,
            secretToken: 'wukui123456'
        )       
    }
    environment{
        CodeRepo="http://gitlab.wukui.com/devops/spring-boot-helloWorld.git"
        HarborServer='ks.harbor.com'
        RegistryUrl="http://ks.harbor.com"
        RegistryCredential='credential-to-harbor_push_image'
        ProjectName='spring-boot-helloworld'
        ImageUrl="${HarborServer}/kubernetes/${ProjectName}"
        ImageTag="${BUILD_ID}"
    }
    stages{
        stage('Source'){
            steps{
                git branch: 'main', credentialsId: 'gitlab-user-jerry-credential', url: "${CodeRepo}"
                
            }
            
        }

        stage('Build'){
            steps{
                container('maven'){
                    sh 'mvn -B -DskipTests clean package'
                }
                
            }
        }
        stage('Test'){
            steps{
                container('maven'){
                     sh 'mvn test'
                }
               
                
            }
            
        }
        /**
        *代码质量评估: 
        *安装sonarqube server,jenkins上安装SonarQube Scanner for Jenkins插件
        *SonarQube Server配置用户,并生成一个token,然后在jenkins上添加认证到sonar的credential(secret text)
        *jenkins的系统配置上添加sonar server的相关信息(SonarQube-Server)
        *配置Jenkins的全局工具sonar-scanner(SonarQube-Scanner-4.8)
        *配置sonar回调jenkins的webhook
        */

        stage("SonarQube Analysis") {
            steps {
                container('maven'){
                    withSonarQubeEnv('SonarQube-Server') {
                        sh 'mvn sonar:sonar'
                    }
                }
                
            }
        }
        stage("Quality Gate") {
            steps {
                timeout(time: 30, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        
        stage('Build Docker Image'){
            steps{
                container('dind'){
                    script{
                        dockerImage = docker.build("${ImageUrl}:${ImageTag}")
                    }
                }
                    
            }
        }
        
        stage('Push Docker Image'){
           
            input{
                message    "Should we continue?"
                ok         "Yes, we should."
                submitter "alice,bob"
                parameters {
                    string(name: 'PERSON', defaultValue: 'Mr Jenkins', description: 'Who should I say hello to?')
                }
                
                    
            }
            when{
                allOf{
                    expression { return params.imagePush}
                }
            }       
            steps{
                container('dind'){
                    script{
                        docker.withRegistry( RegistryUrl, RegistryCredential ) {
                            dockerImage.push()
                            //dockerImage.push('latest')                        
                        } 
                    }
                }
            }       
        }
        stage('Update-manifests') {
        	steps {
        	    container('jnlp') {
        	        sh 'sed -i "s#__IMAGE__#${ImageUrl}:${ImageTag}#g" deploy/all-in-one.yaml'
        	    }  
        	}
        }
        stage('Deploy to prod-env') {
            steps {
                container('kubectl') {
                    withKubeConfig([credentialsId: 'k8s-wukui-cluster-admin', serverUrl: 'https://kubernetes.default.svc']) {
                        sh '''
                            kubectl create namespace hello-prod
                            kubectl apply -f deploy/all-in-one.yaml -n hello-prod
                        '''
                    }
                }
            }
        }
    }
    /**
     *配置构建发送服务: 需要到jenkins的系统配置中(Jenkins Location 添加系统发件人,及其扩展的相关信息)
     *这里仅展示企业微信的配置
     */
    post{
        always{
            qyWechatNotification failNotify: true, webhookUrl: 'https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=df0c4ebb-e0e2-4ed5-829b-0132da2e2e60'
        }

    }
}


