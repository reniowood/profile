---
layout: post
title: "Network flow algorithms"
categories: Algorithm
---

### Flow network

#### Definition

* 각 edge가 capacity와 flow를 가지는 directed graph
    * flow는 capacity를 넘지 않는다.
    * 각 node의 incoming edge의 flow 합과 outgoing edge의 flow 합은 항상 같아야한다.
* source node와 sink node를 가진다.
    * source node는 outgoing edge만을 가진다.
    * sink node는 incoming edge만을 가진다.

### Algorithms

* 다양한 문제를 network flow 문제로 변환하여 풀 수 있다.

#### Maximum flow

* 문제
    * Flow network에서 source로부터 sink로 흘러가는 flow를 만들 때, 가능한 flow의 최대값은?
* 해결 방법
    * Ford-Fulkerson method
        * 아이디어
            * source부터 sink까지 flow를 흘려보낼 수 있는 path(augmenting path)를 찾는다.
            * path가 없으면 종료.
            * path가 있으면 flow를 흘려보낸다.
        * 시간복잡도
            * 알고리즘이 끝난다고 가정하면 (특정 상황에서는 끝나지 않는다), 매번 augmenting path를 찾는 시간은 O(E)에 비례하고, path를 찾았을 때 최소 1의 flow를 흘려보낸다면 최대 f(=maximum flow)번만에 maximum flow를 찾는다. 따라서 시간복잡도는 O(Ef)
    * Edmonds-Karp algorithm
        * Ford-Fulkerson method의 구현
            * augmenting path를 찾기위해 BFS(Breadth-first search)를 사용한다.
                * edge의 갯수가 가장 적은 path를 찾게 된다.
        * 시간복잡도
            * 알고리즘의 매 반복마다 augmenting path를 찾는 시간은 O(E)에 비례한다.
            * augmenting path의 길이는 최대 V다.
            * 한 번의 반복마다 하나의 edge의 flow가 capacity에 도달한다.
            * 따라서 전체 반복 횟수는 O(E)에 비례하고, 매 반복마다 O(VE)에 비례하는 시간이 필요하므로 알고리즘의 시간복잡도는 O(VE^2)가 된다.
        * 그래프 모델링
            * network를 인접 행렬 또는 인접 리스트로 나타낸다.
            * network의 모든 edge에 대해 역방향 edge도 존재함을 표현해야 한다.
                * flow network의 정의에 따라 역방향 edge는 원래 edge flow의 부호를 바꾼 값을 가지므로, 탐색 가능해야 한다.
        * 소스 코드
            * 인접 리스트로 flow network를 표현했다.
        ```java
            public class FlowNetwork {
                private static class Edge {
                    int u, v, capacity, flow;
                    Edge reverse; // 역방향 edge

                    public Edge(int u, int v, int capacity) {
                        this.u = u;
                        this.v = v;
                        this.capacity = capacity;
                        this.flow = 0;
                        this.reverse = null;
                    }
                }

                private int N; // network node의 갯수
                private List[] network; // List<Edge>의 배열. 인접 리스트
                private int source, sink; // source, sink node의 index

                // queue를 이용해 BFS를 실행한다.
                private Edge[] findPath() {
                    Edge[] edges = new Edge[N];
                    Queue<Integer> queue = new LinkedList<>();

                    queue.offer(source);

                    while (!queue.isEmpty()) {
                        int currNode = queue.poll();

                        for (Edge edge : (List<Edge>) network[currNode]) {
                            int nextNode = edge.v;

                            // 아직 capacity가 여유 있는 edge를 찾는다.
                            if (edge.capacity - edge.flow > 0 &&
                                edges[nextNode] == null &&
                                nextNode != source) {
                                    queue.offer(nextNode);
                                edges[nextNode] = edge;
                            }
                        }
                    }

                    return edges;
                }

                public int findMaxFlow() {
                    int maxFlow = 0;

                    while (true) {
                        // BFS를 이용해 augmenting path를 찾는다.
                        // edge[v]는 v를 항햔 edge를 나타낸다. (edge.v == v)
                        Edge[] edges = findPath();

                        if (edges[sink] == null) {
                            break;
                        }

                        // augmenting path에 흐르는 flow 값을 찾는다.
                        int flow = Integer.MAX_VALUE;
                        for (Edge edge = edges[sink]; edge != null; edge = edges[edge.u]) {
                            flow = Math.min(flow, edge.capacity - edge.flow);
                        }

                        // 찾은 flow를 흘려준다.
                        for (Edge edge = edges[sink]; edge != null; edge = edges[edge.u]) {
                            edge.flow += flow;
                            edge.rev.flow -= flow;
                        }

                        // 이번에 찾은 flow를 더해준다.
                        maxFlow += flow;
                    }

                    return maxFlow;
                }
            }
        ```

#### Bipartite matching

* Bipartite graph
    * graph의 node를 두 disjoint set으로 나누었을 때, 모든 edge가 한 집합에서 다른 집합으로 가는 edge라면 그 graph는 bipartite graph라고 한다.
* Bipartite matching
    * graph의 matching은 모든 두 edge 쌍에 대해 공통된 node를 가지지 않은 edge set이다.
    * bipartite matching은 bipartite graph의 matching이다.
* 문제
    * 주어진 bipartite graph의 matching중 가장 많은 edge를 가진 matching의 크기를 구한다.
* 해결 방법
    * Flow network로 모델링해 풀기
        * bipartite graph를 flow network로 모델링해 해당 network의 maximum flow를 찾는 것으로 maximum bipartite matching을 찾는다.
            * 다음과 같이 두 set으로 나뉘어진 bipartite graph가 있을 때, 한 set의 node에는 임의의 source node를 추가하고 capacity가 1인 edge를 연결한다.
            * 마찬가지로, 다른 set의 node에는 임의의 sink node를 추가하고 capacity가 1인 edge를 연결한다.
            * 기존 graph에 있던 edge는 모두 capacity를 1로 만든다.
            * 해당 network의 maximum flow를 구하면 결과값이 곧 maximum bipartite matching의 크기가 된다.

#### Minimum cut

* Cut
    * 어떤 graph의 cut이란 해당 graph의 node를 두 개의 disjoint set으로 나눈 것이다.
    * flow network에서는 source node와 sink node를 기준으로 나눈 두 개의 disjoint set을 의미한다.
* Minimum cut
    * 다양한 graph에서 조금씩 다른 정의의 minimum cut을 사용한다.
    * flow network에서는 cut을 가로지르는 edge의 flow합이 최소인 경우 해당 cut을 minimum cut이라고 한다.
* 문제
    * 임의의 flow network의 minimum cut을 찾는다.
* 해결 방법
    * [Min-cut max-flow theorem](https://en.wikipedia.org/wiki/Max-flow_min-cut_theorem)에 의해 max flow와 min cut은 같게 된다.


#### Minimum vertex cover

* Vertex cover
    * 임의의 graph에 대해 어떤 node의 집합이 있을 때, 모든 edge가 해당 집합에 속한 node와 연결되어 있다면 그 집합을 graph의 vertex cover이라고 한다.
* Minimum vertex cover
    * 한 graph의 모든 vertex cover 중 가장 크기가 작은 vertex cover를 minimum vertex cover라고 한다.
    * 임의의 graph에 대한 minimum vertex cover를 찾는 문제는 NP-hard이다. 다만 bipartite graph에 대해서는 다항 시간 내에 풀 수 있는 해법이 존재한다.
* 문제
    * 임의의 bipartite graph에 대해 minimum vertex cover를 찾는다.
* 해결 방법
    * [Kőnig's theorem](https://en.wikipedia.org/wiki/K%C5%91nig%27s_theorem_(graph_theory))에 의해 bipartite graph의 minimum vertex cover의 크기와 maximum bipartite matching의 크기는 같다.

#### Maximum independent set

* Independent set
    * 임의의 graph에 대해 어떤 node의 집합이 있을 때, 집합의 모든 node가 서로 인접하지 않는다면 그 집합을 graph의 independent set이라고 한다.
* Maximum independent set
    * 한 graph의 모든 independent set 중 가장 크기가 큰 independent set을 maximum independent set이라고 한다.
    * 임의의 graph에 대한 maximum indepent set을 찾는 문제는 NP-hard이다. 다만 bipartite graph에 대해서는 다항 시간 내에 풀 수 있는 해법이 존재한다.
* 문제
    * 임의의 bipartite graph에 대해 maximum independent set을 찾는다.
* 해결 방법
    * [Kőnig's theorem](https://en.wikipedia.org/wiki/K%C5%91nig%27s_theorem_(graph_theory))에 의해 bipartite graph의 maximum independent set의 크기와 minimum vertex cover의 크기는 같다.

#### MCMF(Minimum Cost Maximum Flow)

* 문제
    * Flow network의 각 edge에 cost가 추가되었다. cost는 해당 edge의 flow 1당 비용이다. 이 때, maximum flow의 minimum cost를 구한다.
* 해결 방법
    * Edmonds-Karp algorithm은 augmenting path를 찾을 때 edge의 수가 제일 적은 path를 찾는다. 대신 MCMF를 풀기 위해서는 간선의 수가 아닌 전체 간선의 cost가 최소인 augmenting path를 찾아야 한다. 따라서 BFS가 아닌 최소 거리 알고리즘을 사용해야 한다.
        * 다만 flow network에는 음수 cost를 가진 edge가 있으므로 Bellman-Ford algorithm 혹은 SPFA(Shortest Path Faster Algorithm)을 사용한다.
* 시간복잡도
    * 매번 augmenting path를 찾을 때마다 O(VE)만큼 시간이 비례해 필요하기 때문에 최종 시간복잡도는 O(fVE)다.
* 소스 코드
    * 인접 리스트로 flow network를 표현했다.
    * SPFA를 사용해 min cost augmenting path를 찾았다.
    ```java
            public class FlowNetwork {
                private static class Edge {
                    int u, v, capacity, cost, flow;
                    Edge reverse; // 역방향 edge

                    public Edge(int u, int v, int capacity, int cost) {
                        this.u = u;
                        this.v = v;
                        this.capacity = capacity;
                        this.cost = cost;
                        this.flow = 0;
                        this.reverse = null;
                    }
                }

                private int N; // network node의 갯수
                private List[] network; // List<Edge>의 배열. 인접 리스트
                private int source, sink; // source, sink node의 index

                // queue를 이용해 BFS를 실행한다.
                private Edge[] findMinCostPath() {
                    Edge[] edges = new Edge[N];
                    Queue<Integer> queue = new LinkedList<>();
                    // node가 queue에 들어있는지 O(1)에 확인하기 위한 boolean 배열
                    boolean[] isInQueue = new boolean[N];
                    // source로부터 각 node로의 minimum cost가 들어있는 배열
                    int[] distances = new int[N];
                    Arrays.fill(distances, Integer.MAX_VALUE);

                    queue.offer(source);
                    isInQueue[source] = true;
                    distances[source] = 0;

                    while (!queue.isEmpty()) {
                        int currNode = queue.poll();
                        isInQueue[currNode] = false;

                        for (Edge edge : (List<Edge>) network[currNode]) {
                            int nextNode = edge.v;

                            // 아직 capacity가 여유 있는 edge를 찾는다.
                            if (edge.capacity - edge.flow > 0 &&
                                distances[currNode] + edge.cost < distances[nextNode]) {
                                distances[nextNode] = distances[currNode] + edge.cost;
                                edges[nextNode] = edge;

                                if (!isInQueue[nextNode]) {
                                    queue.offer(nextNode);
                                    isInQueue[nextNode] = true;
                                }
                            }
                        }
                    }

                    return edges;
                }

                public int findMaxFlow() {
                    int maxFlow = 0;

                    while (true) {
                        // BFS를 이용해 augmenting path를 찾는다.
                        // edge[v]는 v를 항햔 edge를 나타낸다. (edge.v == v)
                        Edge[] edges = findMinCostPath();

                        if (edges[sink] == null) {
                            break;
                        }

                        // augmenting path에 흐르는 flow 값을 찾는다.
                        int flow = Integer.MAX_VALUE;
                        for (Edge edge = edges[sink]; edge != null; edge = edges[edge.u]) {
                            flow = Math.min(flow, edge.capacity - edge.flow);
                        }

                        // 찾은 flow를 흘려준다.
                        for (Edge edge = edges[sink]; edge != null; edge = edges[edge.u]) {
                            edge.flow += flow;
                            edge.rev.flow -= flow;
                        }

                        // 이번에 찾은 flow를 더해준다.
                        maxFlow += flow;
                    }

                    return maxFlow;
                }
            }
        ```