# IP Addressing Plan

## CE Network

```text
192.168.10.0/24
```

| Device | Interface | IP Address      |
| ------ | --------- | --------------- |
| CE1    | Gi0/0     | 192.168.10.1/24 |
| CE2    | Gi0/0     | 192.168.10.2/24 |

---

## PE1 - P1

```text
10.10.10.0/30
```

| Device | Interface | IP            |
| ------ | --------- | ------------- |
| PE1    | Gi0/1     | 10.10.10.1/30 |
| P1     | Gi0/0     | 10.10.10.2/30 |

---

## P1 - P2

```text
10.10.10.8/30
```

| Device | Interface | IP             |
| ------ | --------- | -------------- |
| P1     | Gi0/1     | 10.10.10.9/30  |
| P2     | Gi0/0     | 10.10.10.10/30 |

---

## P2 - PE2

```text
10.10.10.12/30
```

| Device | Interface | IP             |
| ------ | --------- | -------------- |
| P2     | Gi0/1     | 10.10.10.13/30 |
| PE2    | Gi0/1     | 10.10.10.14/30 |

---

## Loopbacks

| Device | Loopback | IP             |
| ------ | -------- | -------------- |
| PE1    | Lo1      | 11.11.11.11/32 |
| P1     | Lo1      | 1.1.1.1/32     |
| P2     | Lo1      | 2.2.2.2/32     |
| PE2    | Lo1      | 44.44.44.44/32 |

---

## Important Addressing Lesson

The provider links use separate `/30` networks:

```text
10.10.10.0/30
10.10.10.8/30
10.10.10.12/30
```

Using `/24` on these interfaces would cause overlapping networks.

Example of incorrect configuration:

```text
P1 Gi0/0 = 10.10.10.2/24
P1 Gi0/1 = 10.10.10.9/24
```

Both addresses belong to:

```text
10.10.10.0/24
```

Cisco IOS will therefore report an overlapping subnet.

Correct configuration:

```text
10.10.10.2/30
10.10.10.9/30
```
