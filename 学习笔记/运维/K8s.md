# K8s

## 简介

![image-20211214161857138](C:\Users\12589\AppData\Roaming\Typora\typora-user-images\image-20211214161857138.png)

### 组件

![image-20211214161927226](C:\Users\12589\AppData\Roaming\Typora\typora-user-images\image-20211214161927226.png)![image-20211214162009552](C:\Users\12589\AppData\Roaming\Typora\typora-user-images\image-20211214162009552.png)

![image-20211214161947053](C:\Users\12589\AppData\Roaming\Typora\typora-user-images\image-20211214161947053.png)

### 核心概念

#### pod

- 最小部署单元
- 一组容器集合
- 共享网络
- 短暂生命周期

#### Controller

- 确保预期的pod副本数量
- 无/有状态应用部署
- 确保所有node运行同一个pod

#### Service

- 定义一组pod的访问规则

  

## kubeadm搭建k8s集群

