---
layout: post
title: "BOJ 3938 - Concert Hall Scheduling"
categories: Algorithm
---

[문제 링크](https://www.acmicpc.net/problem/3938)

1년 365일 두 개의 콘서트 홀을 운영하면서 예약 요청 일정과 금액이 주어졌을 때 얻을 수 있는 가장 큰 금액을 묻는 문제이다. 언뜻 보았을 때는 각 예약과 일정을 이분 그래프로 나타낸 후 최소 비용 최대 유량 알고리즘을 사용하면 될 것으로 생각했다. 네트워크 유량 문제가 다 그렇지만 이 문제도 그래프 모델링이 매우 중요하다.

우리가 최대로 만들어야 하는 유량은 곧 예약 숫자이기 때문에 예약을 네트워크에 흐르도록 만들어야 한다. 처음 생각했던 방식으로는 n개의 예약과 365개의 날짜를 예약을 나타내는 간선을 통해 연결해 이분 그래프를 만들어야 한다. 하지만 이렇게 하면 어떻게 해도 예약 일정 하나는 시작일부터 종료일까지 하나의 홀만 사용하고, 부분 예약은 되지 않는다는 제약 조건은 만족할 수 없다.

문제 풀이의 핵심은 아래 예제 데이터에 대해 다음과 같이 유량 네트워크를 모델링하는 것이다.

```
4
1 2 10
2 3 10
3 3 10
1 3 10
```

<img src="/img/algorithm/boj3938/example-network.png" width="500px">

한 번에 최대 두 개의 콘서트 홀에 예약이 가능하므로 1일부터 365일까지 차례대로 용량이 2인 간선으로 연결한다. 그리고 각 예약마다 시작일 정점에서 종료일 다음 날 정점까지 연결하는 용량 1, 비용이 요금에 -1을 곱한 값인 간선을 추가한다. 종료일 정점으로 연결하게 되면 시작일과 종료일이 같은 예약이나 한 예약의 종료일과 다른 예약의 시작일이 같은 것을 처리할 수 없다. 최대 비용을 구해야 하므로 비용도 요금에 -1을 곱한 값을 사용한다. 이 때 365일 정점 다음 정점을 추가해야 한다.

위 네트워크에 최소 비용 최대 유량 알고리즘을 적용한 소스코드는 다음과 같다. 최대 비용을 구해야 하므로 최소 비용을 구한 결과에 -1을 곱해주어야 하는 것을 잊지 말아야 한다.
```java
import java.io.*;
import java.util.*;

public class Main {
    private static final int NUM_DAYS = 365;
    private static final int SOURCE = 0, SINK = NUM_DAYS;

    private static void createNetwork(int n, int[][] applications, int[][] network, int[][] costs) {
        for (int i = 0; i < NUM_DAYS; ++i) {
            network[i][i + 1] = 2;
        }

        for (int i = 0; i < n; ++i) {
            int[] application = applications[i];

            network[application[0]][application[1] + 1] = 1;
            costs[application[0]][application[1] + 1] = -application[2];
            costs[application[1] + 1][application[0]] = application[2];
        }
    }

    private static Integer[] findMinCostPath(int networkSize, int[][] network, int[][] costs) {
        Integer[] parents = new Integer[networkSize];
        Queue<Integer> queue = new LinkedList<>();
        boolean[] isInQueue = new boolean[networkSize];
        int[] distances = new int[networkSize];
        Arrays.fill(distances, Integer.MAX_VALUE);

        queue.offer(SOURCE);
        isInQueue[SOURCE] = true;
        distances[SOURCE] = 0;

        while (!queue.isEmpty()) {
            int curr = queue.poll();
            isInQueue[curr] = false;

            for (int next = 0; next < networkSize; ++next) {
                if (network[curr][next] > 0 && distances[curr] + costs[curr][next] < distances[next]) {
                    distances[next] = distances[curr] + costs[curr][next];
                    parents[next] = curr;

                    if (!isInQueue[next]) {
                        queue.offer(next);
                        isInQueue[next] = true;
                    }
                }
            }
        }

        return parents;
    }

    private static int findMinCostMaxFlow(int networkSize, int[][] network, int[][] costs) {
        int minCost = 0;

        while (true) {
            Integer[] parents = findMinCostPath(networkSize, network, costs);

            if (parents[SINK] == null) {
                break;
            }

            int flow = Integer.MAX_VALUE;
            for (int node = SINK; parents[node] != null; node = parents[node]) {
                flow = Math.min(flow, network[parents[node]][node]);
            }

            for (int node = SINK; parents[node] != null; node = parents[node]) {
                minCost += flow * costs[parents[node]][node];
                network[parents[node]][node] -= flow;
                network[node][parents[node]] += flow;
            }
        }

        return minCost;
    }

    private static int solve(int n, int[][] applications) {
        int networkSize = NUM_DAYS + 1;
        int[][] network = new int[networkSize][networkSize];
        int[][] costs = new int[networkSize][networkSize];

        createNetwork(n, applications, network, costs);

        return -findMinCostMaxFlow(networkSize, network, costs);
    }

    public static void main(String[] args) throws IOException {
        BufferedReader in = new BufferedReader(new InputStreamReader(System.in));

        while (true) {
            int n = Integer.parseInt(in.readLine());

            if (n == 0) {
                break;
            }

            int[][] applications = new int[n][3];

            for (int i = 0; i < n; ++i) {
                StringTokenizer st = new StringTokenizer(in.readLine());

                applications[i][0] = Integer.parseInt(st.nextToken()) - 1;
                applications[i][1] = Integer.parseInt(st.nextToken()) - 1;
                applications[i][2] = Integer.parseInt(st.nextToken());
            }

            System.out.println(solve(n, applications));
        }

        in.close();
    }
}
```