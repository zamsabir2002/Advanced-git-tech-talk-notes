
### Git object with directory structure
```
project/
 ├── a.txt
 ├── b.txt
 ├── src/
 │    ├── main.js
 │    └── utils/
 │          └── helper.js
 └── docs/
       └── guide.md
```

```
Commit C1
  |
  ▼
Tree T_root
  ├── a.txt → Blob A
  ├── b.txt → Blob B
  ├── src → Tree T_src
  │         ├── main.js → Blob M
  │         └── utils → Tree T_utils
  │                     └── helper.js → Blob H
  └── docs → Tree T_docs
            └── guide.md → Blob G
```

<br>
<br>
<br>

| Type       | Purpose             | Contains                                |
| ---------- | ------------------- | --------------------------------------- |
| **blob**   | File contents       | Raw data only (no name, no permissions) |
| **tree**   | Directory snapshot  | Filenames, modes, and pointers          |
| **commit** | Repository snapshot | Metadata + pointer to root tree         |
| **tag**    | Annotated tag       | Metadata + pointer to another object    |


<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>

---

### Git file types and permissions


| Mode       | Type                 | Points To     | When Used        |
| ---------- | -------------------- | ------------- | ---------------- |
| **040000** | Directory (Tree)     | Tree object   | Folders          |
| **100644** | File (normal)        | Blob          | Regular files    |
| **100755** | File (executable)    | Blob          | `chmod +x` files |
| **120000** | Symlink              | Blob          | Symbolic links   |
| **160000** | Submodule entry      | Commit object | Git submodules   |
| **000000** | Deleted entry (temp) | N/A           | Index only       |
