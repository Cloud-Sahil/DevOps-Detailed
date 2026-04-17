# <img src="https://cdn.simpleicons.org/docker" width="40" /> Introduction to Docker

+----------------------+
|    Docker Client     |
|   (CLI / UI)         |
+----------+-----------+
           |
           ▼
+----------------------+
|    Docker Daemon     |
|     (dockerd)        |
+----------+-----------+
           |
   ---------------------------
   |           |             |
   ▼           ▼             ▼
+--------+ +----------+ +----------+
| Images | |Containers| | Networks |
+--------+ +----------+ +----------+
           |
           ▼
+----------------------+
|   Host Operating OS  |
+----------------------+
