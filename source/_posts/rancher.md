---
title: Rancher 常用操作
enlink: rancher-operations
date: 2024-08-30 11:17:23
categories:
- 运维
tags:
- rancher
- docker
---

### 创建 Rancher Server 数据副本
```bash
docker stop lemes-rancher-2.5
docker create --volumes-from lemes-rancher-2.5 --name rancher-data-2023-02-21 rancher/rancher:v2.5.12
```

### 创建备份压缩包
```bash
docker run --volumes-from rancher-data-2023-02-21 -v $PWD:/backup busybox tar zcvf /backup/rancher-data-backup-2023-02-21.tar.gz /var/lib/rancher
```

### 拉去最新镜像
```bash
docker pull rancher/rancher:v2.5.12
docker run -d --restart=unless-stopped \
  --volumes-from rancher-data-backup-2023-02-20 \
  -p 80:80 -p 443:443 \
  --privileged \
  --name=lemes-rancher-2.6 \
  rancher/rancher:v2.5.12

docker run -d --restart=unless-stopped \
  -p 9080:80 -p 8443:443 \
  --privileged \
  --name=rancher \
  rancher/rancher:v2.11.0
```

### 查看k8s环境缺少什么镜像

```bash
docker logs -f kubelet 2>&1 | grep "44:5000"
```

## API

### 升级镜像
```bash
curl -k -X GET -H 'Accept: application/json' -H 'Accept: application/json' -H 'Content-Type: application/json' -H 'Authorization: Bearer token-fpcvv:6k4s8klp5hg9bmdp25x99hgd5hs7s94rlfsxz7pvn2hfp9sp2xdz6m' 'https://10.176.66.20/v3/project/c-b62fg:p-rqhfd/workloads/deployment:default:lemes-auth'

curl -k -X PUT -H 'Accept: application/json' -H 'Accept: application/json' -H 'Content-Type: application/json' -H 'Authorization: Bearer token-fpcvv:6k4s8klp5hg9bmdp25x99hgd5hs7s94rlfsxz7pvn2hfp9sp2xdz6m' -d '{"actions":{"pause":"https://10.176.66.20/v3/project/c-b62fg:p-rqhfd/workloads/deployment:default:lemes-gateway?action=pause","redeploy":"https://10.176.66.20/v3/project/c-b62fg:p-rqhfd/workloads/deployment:default:lemes-gateway?action=redeploy","resume":"https://10.176.66.20/v3/project/c-b62fg:p-rqhfd/workloads/deployment:default:lemes-gateway?action=resume","rollback":"https://10.176.66.20/v3/project/c-b62fg:p-rqhfd/workloads/deployment:default:lemes-gateway?action=rollback"},"annotations":{"cattle.io/timestamp":"2022-04-24T14:29:549+0800"},"baseType":"workload","containers":[{"environmentFrom":[{"optional":false,"source":"field","sourceName":"metadata.name","targetKey":"POD_NAME"},{"optional":false,"source":"field","sourceName":"metadata.namespace","targetKey":"POD_NAMESPACE"},{"optional":false,"source":"configMap","sourceKey":"nacos.addr","sourceName":"lemes-cm","targetKey":"NACOS_ADDR"},{"optional":false,"source":"configMap","sourceKey":"nacos.group","sourceName":"lemes-cm","targetKey":"NACOS_GROUP"},{"optional":false,"source":"configMap","sourceKey":"nacos.namespace","sourceName":"lemes-cm","targetKey":"NACOS_NAMESPACE"},{"optional":false,"source":"configMap","sourceKey":"tz","sourceName":"lemes-cm","targetKey":"TZ"},{"optional":false,"source":"configMap","sourceKey":"java.opts","sourceName":"lemes-cm","targetKey":"JAVA_OPTS"},{"optional":false,"source":"configMap","sourceKey":"java.skywalking","sourceName":"lemes-cm","targetKey":"SKYWALKING"}],"image":"10.176.66.20:5000/lemes-cloud/lemes-gateway:develop-202204241429","imagePullPolicy":"Always","initContainer":false,"livenessProbe":{"failureThreshold":10,"initialDelaySeconds":5,"path":"/actuator/health/liveness","periodSeconds":5,"port":80,"scheme":"HTTP","successThreshold":1,"tcp":false,"timeoutSeconds":10,"type":"/v3/project/schemas/probe"},"name":"lemes-gateway","ports":[{"containerPort":80,"dnsName":"lemes-gateway","hostPort":0,"kind":"ClusterIP","name":"80tcp02","protocol":"TCP","sourcePort":0,"type":"/v3/project/schemas/containerPort"}],"readinessProbe":{"failureThreshold":3,"initialDelaySeconds":5,"path":"/actuator/health/readiness","periodSeconds":5,"port":80,"scheme":"HTTP","successThreshold":1,"tcp":false,"timeoutSeconds":10,"type":"/v3/project/schemas/probe"},"resources":{"type":"/v3/project/schemas/resourceRequirements"},"restartCount":0,"stdin":false,"stdinOnce":false,"terminationMessagePath":"/dev/termination-log","terminationMessagePolicy":"File","tty":false,"type":"/v3/project/schemas/container","volumeMounts":[{"mountPath":"/sidecar","name":"sidecar","readOnly":false,"type":"/v3/project/schemas/volumeMount"}]},{"entrypoint":["cp","-r","/opt/tingyun","/sidecar"],"image":"10.176.66.20:5000/library/tingyun:3.6.1.4","imagePullPolicy":"Always","initContainer":true,"name":"tingyun","ports":[],"resources":{"type":"/v3/project/schemas/resourceRequirements"},"restartCount":0,"stdin":false,"stdinOnce":false,"terminationMessagePath":"/dev/termination-log","terminationMessagePolicy":"File","tty":false,"type":"/v3/project/schemas/container","volumeMounts":[{"mountPath":"/sidecar","name":"sidecar","readOnly":false,"type":"/v3/project/schemas/volumeMount"}]}],"created":"2022-04-12T04:39:45Z","createdTS":1649738385000,"creatorId":null,"deploymentConfig":{"maxSurge":"25%","maxUnavailable":"25%","minReadySeconds":0,"progressDeadlineSeconds":600,"revisionHistoryLimit":10,"strategy":"RollingUpdate"},"deploymentStatus":{"availableReplicas":2,"conditions":[{"lastTransitionTime":"2022-04-12T04:41:16Z","lastTransitionTimeTS":1649738476000,"lastUpdateTime":"2022-04-12T04:41:16Z","lastUpdateTimeTS":1649738476000,"message":"Deployment has minimum availability.","reason":"MinimumReplicasAvailable","status":"True","type":"Available"},{"lastTransitionTime":"2022-04-12T04:39:45Z","lastTransitionTimeTS":1649738385000,"lastUpdateTime":"2022-04-22T06:34:17Z","lastUpdateTimeTS":1650609257000,"message":"ReplicaSet \"lemes-gateway-78f8577b78\" has successfully progressed.","reason":"NewReplicaSetAvailable","status":"True","type":"Progressing"}],"observedGeneration":5,"readyReplicas":2,"replicas":2,"type":"/v3/project/schemas/deploymentStatus","unavailableReplicas":0,"updatedReplicas":2},"dnsPolicy":"ClusterFirst","hostIPC":false,"hostNetwork":false,"hostPID":false,"id":"deployment:default:lemes-gateway","labels":{"app":"lemes-gateway"},"links":{"remove":"https://10.176.66.20/v3/project/c-b62fg:p-rqhfd/workloads/deployment:default:lemes-gateway","revisions":"https://10.176.66.20/v3/project/c-b62fg:p-rqhfd/workloads/deployment:default:lemes-gateway/revisions","self":"https://10.176.66.20/v3/project/c-b62fg:p-rqhfd/workloads/deployment:default:lemes-gateway","update":"https://10.176.66.20/v3/project/c-b62fg:p-rqhfd/workloads/deployment:default:lemes-gateway","yaml":"https://10.176.66.20/v3/project/c-b62fg:p-rqhfd/workloads/deployment:default:lemes-gateway/yaml"},"name":"lemes-gateway","namespaceId":"default","paused":false,"projectId":"c-b62fg:p-rqhfd","publicEndpoints":[{"addresses":["10.122.73.49"],"allNodes":true,"ingressId":"default:lemes-gateway-ig","nodeId":null,"path":"/lemes-api(/|$)(.*)","podId":null,"port":80,"protocol":"HTTP","serviceId":"default:lemes-gateway-svc"},{"addresses":["10.122.73.49"],"allNodes":true,"ingressId":"default:lemes-gateway-ig","nodeId":null,"path":"/lemes-api(/|$)(.*)","podId":null,"port":443,"protocol":"HTTPS","serviceId":"default:lemes-gateway-svc"}],"restartPolicy":"Always","scale":2,"scheduling":{"scheduler":"default-scheduler"},"selector":{"matchLabels":{"app":"lemes-gateway"},"type":"/v3/project/schemas/labelSelector"},"state":"active","terminationGracePeriodSeconds":30,"transitioning":"no","transitioningMessage":"","type":"deployment","uuid":"3afee259-1c17-48c0-8044-19ef85238736","volumes":[{"emptyDir":{"type":"/v3/project/schemas/emptyDirVolumeSource"},"name":"sidecar","type":"/v3/project/schemas/volume"}],"workloadAnnotations":{"deployment.kubernetes.io/revision":"4","kubectl.kubernetes.io/last-applied-configuration":"{\"apiVersion\":\"apps/v1\",\"kind\":\"Deployment\",\"metadata\":{\"annotations\":{},\"labels\":{\"app\":\"lemes-gateway\"},\"name\":\"lemes-gateway\",\"namespace\":\"default\"},\"spec\":{\"replicas\":2,\"selector\":{\"matchLabels\":{\"app\":\"lemes-gateway\"}},\"template\":{\"metadata\":{\"labels\":{\"app\":\"lemes-gateway\"}},\"spec\":{\"containers\":[{\"env\":[{\"name\":\"POD_NAME\",\"valueFrom\":{\"fieldRef\":{\"apiVersion\":\"v1\",\"fieldPath\":\"metadata.name\"}}},{\"name\":\"POD_NAMESPACE\",\"valueFrom\":{\"fieldRef\":{\"apiVersion\":\"v1\",\"fieldPath\":\"metadata.namespace\"}}},{\"name\":\"NACOS_ADDR\",\"valueFrom\":{\"configMapKeyRef\":{\"key\":\"nacos.addr\",\"name\":\"lemes-cm\"}}},{\"name\":\"NACOS_GROUP\",\"valueFrom\":{\"configMapKeyRef\":{\"key\":\"nacos.group\",\"name\":\"lemes-cm\"}}},{\"name\":\"NACOS_NAMESPACE\",\"valueFrom\":{\"configMapKeyRef\":{\"key\":\"nacos.namespace\",\"name\":\"lemes-cm\"}}},{\"name\":\"TZ\",\"valueFrom\":{\"configMapKeyRef\":{\"key\":\"tz\",\"name\":\"lemes-cm\"}}},{\"name\":\"JAVA_OPTS\",\"valueFrom\":{\"configMapKeyRef\":{\"key\":\"java.opts\",\"name\":\"lemes-cm\"}}},{\"name\":\"SKYWALKING\",\"valueFrom\":{\"configMapKeyRef\":{\"key\":\"java.skywalking\",\"name\":\"lemes-cm\"}}}],\"image\":\"10.176.66.20:5000/lemes-cloud/lemes-gateway:develop-202204111455\",\"imagePullPolicy\":\"Always\",\"livenessProbe\":{\"failureThreshold\":10,\"httpGet\":{\"path\":\"/actuator/health/liveness\",\"port\":80},\"initialDelaySeconds\":5,\"periodSeconds\":5,\"timeoutSeconds\":10},\"name\":\"lemes-gateway\",\"ports\":[{\"containerPort\":80}],\"readinessProbe\":{\"httpGet\":{\"path\":\"/actuator/health/readiness\",\"port\":80},\"initialDelaySeconds\":5,\"periodSeconds\":5,\"timeoutSeconds\":10},\"volumeMounts\":[{\"mountPath\":\"/sidecar\",\"name\":\"sidecar\"}]}],\"initContainers\":[{\"command\":[\"cp\",\"-r\",\"/opt/tingyun\",\"/sidecar\"],\"image\":\"10.176.66.20:5000/library/tingyun:3.6.1.4\",\"imagePullPolicy\":\"Always\",\"name\":\"tingyun\",\"volumeMounts\":[{\"mountPath\":\"/sidecar\",\"name\":\"sidecar\"}]}],\"volumes\":[{\"emptyDir\":{},\"name\":\"sidecar\"}]}}}}"},"workloadLabels":{"app":"lemes-gateway"}}
' 'https://10.176.66.20/v3/project/c-b62fg:p-rqhfd/workloads/deployment:default:lemes-auth'
```


