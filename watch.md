<a href="https://kubernetes.io/zh-cn/docs/reference/using-api/api-concepts/" target="_blank">watch 检测变更 🪂</a>

/api-gateway/kubernetes/calico3/apis/asm.alauda.io/v1beta3/namespaces/testhl/microservices/asm-cal3  
/api-gateway/kubernetes/calico3/apis/asm.alauda.io/v1beta3/namespaces/testhl/microservices?fieldSelector=metadata.name=asm-cal3  
/api-gateway/kubernetes/calico3/apis/asm.alauda.io/v1beta3/namespaces/testhl/microservices?fieldSelector=metadata.name=asm-cal3&watch=true&allowWatchBookmarks=true&timeoutSeconds=59&resourceVersion=5937358  

- K8S api 允许通过 watch 来进行更改追踪
- watch 查询的是整个资源列表，但是可以采用 `fieldSelector` ([fieldSelector🪂](https://kubernetes.io/zh-cn/docs/concepts/overview/working-with-objects/field-selectors/) k8s 字段选择器(过滤器)，可以选择一个或者对个资源字段的值) 过滤特定的资源。
- 在首次 `watch` 请求之后，API 服务器会响应一个 `resourceVersion` 字段(k8s 资源对象本身有)，在下一次资源 `watch` 时，url 中会带上这个 `resourceVersion` ，服务器响应这个 `resourceVersion` 之后的变更。除了首次发送的 `watch` 请求，之后的 `watch` 预览中无法查看返回的数据，除非数据有更改，同时更改会显示在之前的长连接里
- 长连接的持续时间在 60s 左右 (标头 connection: keep-alive)

---

查找详情页网络请求遇到的问题：
一直查询的是 `watchResource` 发送的长连接对应的网络请求。设置轮询之后，这里刷新页面不是复用之前的长连接更新数据 (没有设置轮询时，`http` 环境下 `watchResource` 还是会以长连接的形式发送网络请求)

```js[1|2-4|5]
 /**
   * 监听业务集群单个资源对象
   */
  watchResource<T extends KubernetesResource>(
    params: WatchErebusResourceParams<R>,
  ) {
    return DOWNGRADE_WATCH_ENABLED && params.downgradePolling
      ? createPollObservable<T>(
          this._getResource<T>(params),
          this._getDowngradePolling(params),
        )
      : this._watchResource<T>(params);
  }
```

---

同时这里也涉及如何标准资源对应的 `api`，标准 `API` 前缀有 `/api/` (`Global` 集群 `API Group` 为 `core` 的资源), `/apis/` (`Global` 集群 `API Group` 不是 `core` 的资源), `/kubernetes/` (业务集群下的资源),资源列表中的具体资源在资源类型后会带上名字，如 `/microservices/asm-cal3`
