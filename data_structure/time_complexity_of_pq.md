# java.util.PriorityQueue 의 시간복잡도

>Implementation note:
this implementation provides O(log(n)) time for the enqueing and dequeing methods (offer, poll, remove() and add);
linear time for the remove(Object) and contains(Object) methods; and constant time for the retrieval methods (peek, element, and size).

from Java API Specification

pq는 대부분 heap으로 구현되어
**add** -> 마지막 노드에 임의 삽입 후 부모와의 비교 과정에 O(log(n))
**poll** -> 마지막 노드가 root 로 올라와 자식과의 swap 을 거치는데 O(log(n))
**peek,element,size** -> 따로 기록해두면 돼서 O(1) 으로 구현 가능하다
**contains** -> 별도 배열에 기록해두면 O(n) 으로 존재 여부를 파악할 수 있다.

**remove(Object)**
 -> contains 확인에 O(n) 타임을 소모하고, 포함시에 별도 삭제 최소heap에 추가해둔다
pop 연산이 들어왔을 때 삭제힙과 본 힙의 top 이 같은 경우 무시하며 pop 해주면 O(n) 레벨에 삭제 가능,
여기서 contains 연산이 hash 등으로 구현되어있다면 O(log(n)) 레벨에도 삭제 연산이 가능할 것이다.
