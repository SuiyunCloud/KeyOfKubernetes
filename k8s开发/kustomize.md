### kustomize

#### 简介

kustomize 是一个通过 kustomization 文件定制 kubernetes 对象的工具，它可以通过一些资源生成一些新的资源，也可以定制不同的资源的集合。

一个比较典型的场景是有一个应用，在不同的环境例如生产环境和测试环境，它的 yaml 配置绝大部分都是相同的，只有个别的字段不同，这时候就可以利用 kustomize 来解决，kustomize 也比较适合用于 gitops 工作流。

<img src="/Users/cloud/Knowledge/22CloudNative/02K8sNotes/pics/kustomize.png" alt="img" style="zoom: 50%;" />
如上图所示，有一个 ldap 的应用，/base目录保存的是基本的配置，/overlays里放置的不同环境的配置，例如 /dev、/staging，/prod这些就是不同环境的配置，/base等文件夹下都有一个 kustomization .yml 文件，用于配置。

执行 kustomize build dir的方式就可以生成我们最后用于部署的 yaml 文件，也就是进行到了我们上图的第四步，然后通过 kubectl apply -f命令进行部署。

#### 布局

```markdown
├── base
│   ├── deployment.yaml
│   ├── kustomization.yaml
│   └── service.yaml
└── overlays
    ├── dev
    │   ├── kustomization.yaml
    │   └── patch.yaml
    ├── prod
    │   ├── kustomization.yaml
    │   └── patch.yaml
    └── staging
        ├── kustomization.yaml
        └── patch.yaml
```

一个常见的项目 kustomize 项目布局如上所示，可以看到每个环境文件夹里面都有一个 kustomization.yaml 文件，这个文件里面就类似配置文件，里面指定源文件以及对应的一些转换文件，例如 patch 等

##### kustomization.yml

一个常见的 kustomization.yml 如下所示，一般包含 apiVsersion 和 kind 两个固定字段

```yaml
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

resources:
- manager.yaml

configMapGenerator:
- files:
  - controller_manager_config.yaml
  name: manager-config
```

kustomize 提供了比较丰富的字段选择，除此之外还可以自定义插件，下面会大概列举一下每个字段的含义，当我们需要用到的时候知道有这么个能力，然后再去 Kustomize 官方文档 (https://kubectl.docs.kubernetes.io/zh/guides/) 查找对应的 API 文档就行了

> resources: 表示 k8s 资源的位置，这个可以是一个文件，也可以指向一个文件夹，读取的时候会按照顺序读取，路径可以是相对路径也可以是绝对路径，如果是相对路径那么就是相对于 kustomization.yml的路径
>
> crds: 和 resources 类似，只是 crds 是我们自定义的资源
>
> namespace: 为所有资源添加 namespace
>
> images: 修改镜像的名称、tag 或 image digest ，而无需使用 patches
>
> replicas: 修改资源副本数
>
> namePrefix: 为所有资源和引用的名称添加前缀
>
> nameSuffix: 为所有资源和引用的名称添加后缀
>
> patches: 在资源上添加或覆盖字段，Kustomization 使用 patches 字段来提供该功能。
>
> patchesJson6902: 列表中的每个条目都应可以解析为 kubernetes 对象和将应用于该对象的 JSON patch。
>
> patchesStrategicMerge: 使用 strategic merge patch 标准 Patch resources.
>
> vars: 类似指定变量
>
> commonAnnotations: 为所有资源加上 annotations 如果对应的 key 已经存在值，这个值将会被覆盖

```markdown
commonAnnotations:
  app.lailin.xyz/inject: agent

resources:
- deploy.yaml
```

> commonLabels: 为所有资源的加上 label 和 label selector 注意：这个操作会比较危险

```markdown
apiVersion: kustomize.config.k8s.io/v1beta1
kind: Kustomization

commonLabels:
  app: bingo
```

> configMapGenerator 可以生成 config map，列表中的每一条都会生成一个 configmap
>
> secretGenerator 用于生成 secret 资源
>
> generatorOptions 用于控制 configMapGenerator 和 secretGenerator 的行为