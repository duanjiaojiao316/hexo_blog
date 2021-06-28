---
title: BFS算法框架-解开密码锁的最少次数
categories:
  - 算法
tags:
  - BFS（广度优先搜索）
translate_title: bfs-algorithm-frameworkthe-minimum-number-of-times-to-unlock-the-password-lock
date: 2020-12-15 20:25:17
---


遍历所有的密码

```java
// 密码锁向上拨动
String plusOne(String secret, int i) {
    char[] ch = secret.toCharArray();
    if(ch[j] == '9'){
        ch[j] = '0';
    } else {
        ch[j] += 1;
    }
    return String(ch);
}

// 密码锁向上拨动
String downOne(String secret, int i) {
    char[] ch = secret.toCharArray();
    if(ch[j] == '0'){
        ch[j] = '9'
    } else {
        ch[j] -= 1;
    }
    return String(ch)
}

// 打印出所有的密码组合
void BFS(String target) {
    // 队列
    Queue[String] queue = new Linkedlist<>();
    queue.offer("0000");
    
    while(!queue.empty()){
        int size = queue.size;
        for(int i = 0; i < size; i++) {
           String cur = queue.poll();
            // 这里判断是否达到终点
            System.out.println(cur);

            for (int j = 0; j < 4; j++) {
                String up = plusOne(cur, j);
                String down = downOne(cur, j);
                q.offer(up);
                q.offer(down);
            } 
        }
        // 这里可以进行步长计算
    }
    return
}
```

以上代码由于没有终止条件，`0000`拨动成`0001`，同时`0001`也可以拨动成`0000`



加入死亡密码和终止密码

```java
int BFS(String target, String[] deadends) {
    // 死亡密码
    Set[String] deadSecrets = new HashSet<>();
    for(String d: deadends) {
        deadSecrets.add(d);
    }
    
    Set[String] visited = new HashSet<>();
    // 队列
    Queue[String] queue = new Linkedlist<>();
    // 初始化步长
    int step = 0;
    queue.offer("0000");
    visited.add("0000");
    
    while(!queue.empty()){
        
        int size = queue.size();
        for (int i = 0; i < size; i++) {
            String cur = queue.poll();
            // 判断是否是死亡密码
            if (deadends.contains(cur)) {
                continue;
            }
             // 这里判断是否达到终点
            if (cur.equal(target)) {
                return step;
            }
            System.out.println(cur);

            for (int j = 0; j < 4; j++) {
                String up = plusOne(cur, j);
                if (!visited.contains(up)){
                    q.offer(up);
                    visited.add(up);
                }
                
                String down = downOne(cur, j);
                if (!visited.contains(up)){
                    q.offer(down);
                    visited.add(down);
                }
               
            }
        }
        step ++;
        // 这里可以进行步长计算
    }
    //如果遍历都没有查找到目标，证明没有办法到目标密码
    return -1;
}
```

传统的 BFS 框架就是从起点开始向四周扩散，遇到终点时停止；而双向 BFS 则是从起点和终点同时开始扩散，当两边有交集的时候停止。

不过，双向 BFS 也有局限，因为你必须知道终点在哪里。比如二叉树最小高度的问题，一开始根本就不知道终点在哪里，也就无法使用双向 BFS；但是密码锁的问题，可以使用双向 BFS 算法来提高效率



```java
int BFS(String target,String[] deadends) {
     // 死亡密码
    Set[String] deadSecrets = new HashSet<>();
    for(String d: deadends) {
        deadSecrets.add(d);
    }
    
    Set[String] visited = new HashSet<>();
    // 队列
    Set[String] queue1 = new HashSet<>();
    Set[String] queue2 = new HashSet<>();
    // 初始化步长
    int step = 0;
    queue1.add("0000");
    queue2.add(target);
    visited.add("0000");
    visited.add(target);
    
    while(!queue1.empty() && !queue2.empty()) {
        // 哈希集合在遍历的过程中不能修改，用 temp 存储扩散结果
        Set<String> temp = new HashSet<>();

        /* 将 q1 中的所有节点向周围扩散 */
        for (String cur : q1) {
            /* 判断是否到达终点 */
            if (deadSecrets.contains(cur))
                continue;
            if (q2.contains(cur))
                return step;
            visited.add(cur);
            /* 将一个节点的未遍历相邻节点加入集合 */
            for (int j = 0; j < 4; j++) {
                String up = plusOne(cur, j);
                if (!visited.contains(up))
                    temp.add(up);
                String down = downOne(cur, j);
                if (!visited.contains(down))
                    temp.add(down);
            }
        }
        // 增加步长
        step ++;
        // 这里交换queue1和queue2 下一个循环就会扩展queue2相邻的密码
        queue1 = queue2;
        queue2 = temp;
    }
    return -1;
}
```