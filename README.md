## CICD--Test

主要演示了Freestyle 流水线过渡到Pipeline的过程，中间涉及到了较多的插件和围绕DevOps相关的工具

### 测试环境简要说明

+ Jenkins Master独立于集群外部
+ gitlab独立于集群外
+ SonarQube独立于集群外
+ Harbor独立于集群外
+ Jenkins Agent 运行于k8s环境中,请[参考](https://github.com/ForcemCS/CICD/Jenkin Agent.txt)

+ Rollouts
+ ArgoCD