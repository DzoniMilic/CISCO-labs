# Topology

## Physical Connections

```text
CE1 ---------------- PE1 ---------------- P1 ---------------- P2 ---------------- PE2 ---------------- CE2
     Gi0/0     Gi0/2      Gi0/1     Gi0/0      Gi0/1     Gi0/0      Gi0/1     Gi0/2      Gi0/0
```

### Connections

| Device A | Interface | Device B | Interface |
| -------- | --------- | -------- | --------- |
| CE1      | Gi0/0     | PE1      | Gi0/2     |
| PE1      | Gi0/1     | P1       | Gi0/0     |
| P1       | Gi0/1     | P2       | Gi0/0     |
| P2       | Gi0/1     | PE2      | Gi0/1     |
| PE2      | Gi0/2     | CE2      | Gi0/0     |

## Logical Roles

| Router | Role          |
| ------ | ------------- |
| CE1    | Customer Edge |
| PE1    | Provider Edge |
| P1     | Provider Core |
| P2     | Provider Core |
| PE2    | Provider Edge |
| CE2    | Customer Edge |

## Important

CE-facing PE interfaces:

```text
PE1 Gi0/2
PE2 Gi0/2
```

are used as Layer 2 attachment circuits for xConnect.

They do not require IP addresses.
