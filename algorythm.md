# 노드 역순 정렬

```python

def reverse(self):
        prev = None
        node = self.head
        while node:
            save = node.next
            node.next = prev
            prev = node
            node = save
            
        self.head = prev   
```