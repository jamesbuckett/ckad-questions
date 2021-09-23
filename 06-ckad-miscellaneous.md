## Sample CKAD Extension Questions and Answers

#### 01. Sample Question.

<details><summary>show</summary>
<p>

```bash
Sample

```

</p>
</details>

#### 06-01. List all the Kubernetes resources that can be found inside a namespace. By name only.

<details><summary>show</summary>
<p>

kubernetes.io: [Not All Objects are in a Namespace](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/#not-all-objects-are-in-a-namespace)

```bash
kubectl api-resources --namespaced=true | more   
```

Output:
```bash
NAME                               SHORTNAMES                           APIVERSION                                  NAMESPACED   KIND
bindings                                                                v1                                          true         Binding
configmaps                         cm                                   v1                                          true         ConfigMap
endpoints                          ep                                   v1                                          true         Endpoints
...

# Do not need the additional supplied columns.

```

</p>
</details>

<details><summary>show</summary>
<p>

```bash
clear
kubectl api-resources --namespaced=true -o name | more
```

Output:
```bash
bindings
configmaps
endpoints
events
...
```

</p>
</details>

*End of Section*