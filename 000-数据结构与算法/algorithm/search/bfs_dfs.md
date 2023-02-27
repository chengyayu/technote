# 深度和广度优先搜索

> 算法是作用于具体数据结构之上的。

深度优先搜索算法和广度优先搜索算法都是基于「图」这种数据结构的。「树」可以视为特殊的图。

## 广度优先算法（BFS）

```go
func BFS(g Graph, start int, visit func(vertex *Vertex) bool) error {
    startVertex := g.getVertex(start)
    if startVertex == nil {
        return fmt.Errorf("invalid vertex key : %v", start)
    }

    // 初始化队列和已访问集合
    queue := []*Vertex{startVertex}
    visited := make(map[*Vertex]struct{})

    // 循环遍历直到队列为空
    for len(queue) > 0 {

        // 出队一个元素
        curVertex := queue[0]
        queue = queue[1:]

        // 已处理过的直接跳过
        if _, ok := visited[curVertex]; ok {
            continue
        }

        // 处理并标记已处理
        visited[curVertex] = struct{}{}
        if stop := visit(curVertex); stop {
            break
        }

        // 将当前节点的下一层顶点入队
        queue = append(queue, curVertex.adjacent...)
    }

    return nil
}
```

## 深度优先算法（DFS）


## Letcode Problems

- [二叉树的层次遍历](https://leetcode.cn/problems/binary-tree-level-order-traversal/#/description)
