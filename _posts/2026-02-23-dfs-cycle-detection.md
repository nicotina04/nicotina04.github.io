유향 그래프에서 사이클을 찾는 방법을 알아보자.

왜 유향 그래프라 했냐면 무향 그래프에는 별도의 정의가 있지 않는 한 서로를 왕복할 수만 있으면 사이클이 존재하는 것이기 때문이다.

## DFS 스패닝 트리와 간선 분류

어떤 그래프를 DFS로 탐색했을 때 지나간 엣지만을 남겨놓으면 트리 형태가 되는데 이를 DFS 스패닝 트리라고 한다.

어떤 그래프를 DFS 스패닝 트리로 변환했을 때 그래프의 모든 엣지를 4가지로 분류할 수 있다.

#### Tree edge (트리 간선)

트리 간선은 DFS 스패닝 트리에 속하는 간선이다. 즉, DFS를 하면서 거쳐간 간선이다.

#### Forward edge (순방향 간선)

DFS를 통해 어떤 연결된 두 노드 사이의 부모 자식 관계를 정립할 수 있지만 DFS 스패닝 트리에서 트리 간선은 아닌 간선이다.

#### Back edge (역방향 간선)

DFS 스패닝 트리에서 자식 -> 부모로 거슬러 올라가는 간선이다.

#### Cross edge (교차 간선)

위 세종류를 제외한 나머지 간선들이며 형제 노드끼리 연결된 간선이다.

각 간선들은 DFS의 구현에 따라 언제든지 그 지위가 바뀔 수 있다.

## 방향 그래프에서 사이클 찾기

위 분류를 통해 방향 그래프에서 사이클이 존재하는 경우를 간단하게 정의할 수 있다.

바로 역방향 간선의 존재 여부이다. 역방향 간선의 개수는 곧 사이클의 개수가 된다.

그러면 역방향 간선을 어떻게 찾아야 할까? DFS를 응용하면 간단하게 구현할 수 있다.

우선 다음 두 정보를 관리하는 배열을 둔다.

discovered\[\] : 기존의 DFS에서 visited\[\]의 역할을 하는 그 배열이다. 해당 노드를 방문했는지 여부를 관리한다.

finished\[\] : 해당 노드를 대상으로 호출한 함수가 종료되었는지 여부를 관리하는 배열이다.

DFS 탐색을 하면서 방문하지 않은 노드는 방문하고 만약 해당 노드가 방문된 상태인데 종료가 되지 않았다면(discovered == true && finished == false이면) 역방향 간선을 찾았다는 의미이다. 다음 그림을 보자.

![Directed Graph](/assets/img/posts/2026-02-23-dfs-cycle-detection/1.png)

위 그래프의 경우 방문된 노드를 재확인할 일이 없으므로 사이클이 없다.

![Stack Trace in dfs()](/assets/img/posts/2026-02-23-dfs-cycle-detection/2.png)

위 그래프의 경우 dfs(1), dfs(2), dfs(3)이 차례로 호출된 상태에서 dfs(3)이 1번 노드를 접근하려고 하고 있다.

이때 1번 노드에 방문은 한 상태지만 dfs(1)의 호출은 아직 끝나지 않은 상태이므로 finished\[1\] == false이며 back edge 판정이 되고 이 그래프는 사이클을 가짐을 알 수 있다.

다음은 위 설명을 코드로 구현한 예시이다.

```
int discovered[101];
bool finished[101];
vector<int> graph[101];
int node_order;
int cycle;

void dfs(int node)
{
  // 해당 노드가 몇번째에 방문되었는지 기록
  discovered[node] = node_order++;

  for (int i: graph[node])
  {
    if (discovered[i] == -1)
      dfs(i);
    else if (!finished[i])
      ++cycle;
    // else는 cross edge인 경우
  }

  // 해당 노드의 함수 호출이 종료되었음을 명시
  finished[node] = true;
}
```

그러면 그래프 사이클을 찾는 문제를 몇 개 풀어보자.

[https://www.acmicpc.net/problem/16724](https://www.acmicpc.net/problem/16724)

조건을 만족하는 안전 구역이 최소 몇개가 필요한지 묻고 있다.

조금만 생각해보면 지도 밖을 나가는 경우가 없다는 것은 그래프에 사이클이 존재하여 어떻게 이동하든 사이클에 휘말린다는 뜻이 된다.

즉 회원들을 지키기 위해서는 사이클을 끊어야 하며 몇 개의 사이클을 끊어야 하는지 사이클의 개수를 찾는 문제로 환원할 수 있다.

2차원 배열에 대해 사이클의 갯수를 찾아 출력하면 정답을 받을 수 있다.

다음은 ICPC Korea regional 기출인 텀 프로젝트이다.

[https://www.acmicpc.net/problem/9466](https://www.acmicpc.net/problem/9466)

이 문제는 위와는 반대로 사이클을 생성할 수 없는 노드의 개수를 찾는 문제이다.

이 문제를 보면 위의 예시 코드에서 왜 discovered에 방문 순서를 기록했는지 알 수 있다.

back edge를 찾게 될 때 현재 방문한 노드의 번호에서 사이클을 마주친 노드의 방문 번호를 빼면 사이클에 속한 노드의 개수가 되기 때문이다.

아래 코드처럼 예시를 조금만 수정하여 간단하게 정답을 받을 수 있다.

```cpp
#pragma GCC target("avx,avx2,fma")
#pragma GCC optimize("Ofast")
#pragma GCC optimize("unroll-loops")

#include <bits/stdc++.h>

using namespace std;

int discovered[100001];
bool finished[100001];
int next_node[100001];
int node_order;
int cycle;
int grouped;
int t;

void dfs(int node)
{
  discovered[node] = node_order++;

  if (discovered[next_node[node]] == -1)
    dfs(next_node[node]);
  else if (!finished[next_node[node]])
    grouped += discovered[node] - discovered[next_node[node]] + 1;

  finished[node] = true;
}

int main()
{
  ios::sync_with_stdio(false);
  cin.tie(nullptr);
  cin >> t;

  while (t--)
  {
    node_order = 0;
    grouped = 0;
    memset(discovered, -1, sizeof discovered);
    memset(finished, 0, sizeof finished);
    int n;
    cin >> n;

    for (int i = 1; i <= n; ++i)
      cin >> next_node[i];
    
    for (int i = 1; i <= n; ++i)
      if (!finished[i])
        dfs(i);
    cout << n - grouped << '\n';
  }
}
```

지금까지 방향 그래프에서 사이클을 찾는 방법을 알아보았다.

사이클 찾기는 고급 그래프 알고리즘에서 응용되는 경우가 심심찮게 보이니 기본기를 확실히 익히면 도움이 될 것이다.