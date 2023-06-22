方式一:以静态容器(iamge: jenkins/inbound-agent)的方式作为代理节点(容器的创建可以借助docker compose)
1.首先打开jenkins master的TCP端口(默认50000),然后勾选java web协议
2.jenkins master上添加节点(name--执行器数量--远程工作目录(/home/jenkins/agent)--标签(linux-jnlp-agent linux-agent) \
启动方式(java web)) 
3.将2获得的secret传递到jenkins/inbound-agent中(其他的环境变量参阅dockerhub)
4.也可以为master设置一个标签

方式二:在具有docker engine的主机之上动态的创建agent
在Master主机上安装插件(docker docker pipeline),但是各个agent不共享工作空间,所以会存在一定的问题

方式三: 在k8s集群上创建pod.用作agent
1.Master安装插件(kubernetes plugin)
2.节点管理--新建一个类型为kubernetes的cloud
3.配置kubernetes Cloud details(Kubernetes 地址---Kubernetes 服务证书 key---Kubernetes 命名空间---凭据 \
  可以通过RBAC的相关知识,创建一个服务账号，当然也可以获取集群已经存在的sa的token---jenkins地址(http://jenkins.wukui.com:8080)---jenkins通道(12.0.0.5:50000))
4.添加pod模板,创建pod，以便用来运行不同的stage.多个模板可以实现不同功效的agent
5.一个Pod模板中可以运行多个cintainer，以便实现不同的stage,而且他们的存储是共享的
5.需要注意的是当k8s集群上的agent联系外部的服务时,可以通过修改coredns的配置文件来实现
6.更重要的是,我们不应该在pod中的某一个container内来安装各种工具来实现job,而是通过不同的image来实现不同的stage
7.在satge中若没有明确指定是在哪个container运行的，则是使用默认的lnlp容器来完成的
8.其中有一个特殊的容器镜像可以使用，那就是bind,这个镜像可以认为提供了dokcer的客户端程序，可以利用宿主机的docker engine来实现相关的功能，例如build push 等
9.还有一个容器就是可以使用kubectl命令，把应用部署到目标kubernetes集群（可以是当前集群也可以是其他集群）
10.假设现在有一个pod模板只有maven container，那么另一个模板就可以继承这个模板，也就增加了一个maven container
11.在构建镜像的时候，除了可以在Pod中的这个容器(image: docker:cli)内直接使用shell 形式的docker命令外 \
   还可以借助docker pipeline插件来完成docker 镜像的相关操作（可以简单的理解为调用函数的某个方法来实现）
12.再新增一个pod模板，运行kubectl
13.我们需要有一个kubectl对接的目标集群，可能需要认证，授权
14.kubectl container从credential获取信息需要安装插件kubernetes credentials  cli
15.有关认证的内容配置参考https://learnk8s.io/authentication-kubernetes#granting-access-to-the-cluster-to-external-users \
   这里使用静态令牌认证，假设现在有一个csv文件 格式为wukui123456,wukui,1001,"kube-admins" \
   在利用RBAC进行授权即可（kubectl create   clusterrolebinding wukui-cluster-admin --clusterrole=cluster-admin   --user=wukui （权限较大））
16.然后在jenkins主机上配置属于wukui的 secret text
17.更需要注意的是在通过pipeline部署应用的时候，一定要对返回的结构进行判断，以便自动处理可能预见的问题
18.构建应用通过之后，我们利用Pipeline From SCM


二.借助ArgoCD实现pull模式的pipeline
1.在gitlab上新建一个空的仓库（配置仓库）
2.编写好yaml或者其他argocd支持的配置格式的文件推送到仓库
3.删除push流水线中的CD任务
4.可以在新增一个任务，当push任务成功之后，触发下一个修改配置仓库的任务
5.然后借助argocd crontroller完成CD任务
