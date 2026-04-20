# PES-VCS — Version Control System

Name: Vijay Suriya P  
SRN: PES1UG24AM320  
Platform: Ubuntu 22.04  

---

## Objective

Build a local version control system that tracks file changes, stores snapshots efficiently, and supports commit history. The system is inspired by Git and uses content-addressable storage.

---

## Features Implemented

- Content-addressable object storage (blobs, trees, commits)
- Directory tree snapshots
- Staging area (index)
- Commit creation and history tracking
- Log traversal

---

## Commands Implemented

- pes init — Initialize repository  
- pes add <file> — Stage files  
- pes status — Show file status  
- pes commit -m "msg" — Create commit  
- pes log — Show commit history  

---

## Repository Structure

.pes/
├── objects/
├── refs/
│   └── heads/
├── index
└── HEAD

---

## Analysis Questions

### Q5.1 — pes checkout

To implement pes checkout <branch>:
- Update .pes/HEAD to point to the branch (ref: refs/heads/<branch>)
- Read the branch’s commit and obtain its tree
- Recursively walk the tree and write each file to the working directory
- Delete files that are not present in the target tree
- Update .pes/index to match the new tree

If the branch does not exist, create .pes/refs/heads/<branch> with the current HEAD commit hash.

This operation is complex due to recursive directory handling, file updates/deletions, and conflict detection.

---

### Q5.2 — Dirty working directory detection

For each IndexEntry:
- Call stat() on the file
- If mtime or size differs → file is modified
- If file is missing → it is deleted

Then compare with the target branch’s tree:
- If the same file has a different hash in the target tree and is modified locally → conflict

In such cases, checkout must be aborted to prevent overwriting changes.

---

### Q5.3 — Detached HEAD

In detached HEAD state, .pes/HEAD stores a commit hash instead of a branch reference.

New commits are created normally, but they are not referenced by any branch and can become unreachable when switching branches.

Recovery:
- Get the commit hash using: cat .pes/HEAD
- Create a new branch:
  echo "<hash>" > .pes/refs/heads/recovery
- Attach HEAD:
  echo "ref: refs/heads/recovery" > .pes/HEAD

---

### Q6.1 — Garbage Collection

Use a mark-and-sweep algorithm:

Mark phase:
- Traverse all objects reachable from branch tips
- Follow commits → trees → blobs
- Store hashes in a HashSet

Sweep phase:
- Iterate through .pes/objects/
- Delete objects not present in the reachable set

Data structure: Hash set (O(1) lookup)

Complexity:
For ~700,000 objects → O(N), linear time

---

### Q6.2 — GC Race Condition

Race condition:
- GC completes marking reachable objects
- A concurrent commit writes a new blob
- GC deletes the blob (not in reachable set)
- Commit references deleted blob → corruption

Git avoids this using:
- Grace period (objects are not deleted immediately)
- Reflog (recent history treated as reachable)
- GC locking (gc.pid)
- Writing objects before updating references

---

## Conclusion

This project demonstrates how a version control system works internally using filesystem concepts such as hashing, atomic writes, directory structures, and reference management.
