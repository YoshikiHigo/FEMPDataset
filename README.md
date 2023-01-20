# FEMPDataset

FEMPDataset is a dataset of functionally equivalent (in short, FE) method pairs.
FEMPDataset include XXX FE method pairs that have been checked by three programmers.


## Quick start

First of all, please make sure that SQLite is installed in your environment.
```shell-session
$sqlite3 ijadataset.db
SQLite version 3.39.5 2022-10-14 20:58:05
Enter ".help" for usage hints.
sqlite>
```

### Tables in ijadataset.db

There are three tables `methods`, `pairs`, and `verifiedpairs` in ijadataset.db.
```shell-session
sqlite> .tables
methods        pairs          verifiedpairs
```

Of those three tables, table `verifiedpairs` includes information on FE method pairs.


### Table `verifiedpairs`

The schema of table `verifiedpairs` is as follows.

```shell-session
sqlite> .schema verifiedpairs
CREATE TABLE verifiedpairs(pairid integer, reviewera integer, reviewerb integer, reviewerc integer, consensus integer, reason blob);
```

- `pairid` means the unique identifier for the method pair.
- `reviewera`, `reviewerb`, and `reviewerc` mean that they represent the judgement results that were individually confirmed by each reviewer. `1` means functionally equivalent and `0` means not functionally equivalent.
- `consensus` means the final decision result. If all three reviewers gave `1` or `0`, then `consensus` is equal to that value. If there was a difference between the three reviewers' judgements, they had a discussion about the method pair, and `consensus` represent the result of that discussion.

You can get the number of FE method pairs that have been validated by the three reviewers with the following command.

```shell-session
sqlite> select count(*) from verifiedpairs where consensus = 1;
select count(*) from verifiedpairs where consensus = 1;
1342
```

The following command enables you to see the source code of FE method pairs that have been validated by the three reviewers.

```shell-session
sqlite> select (select rtext from methods where id = (select leftMethodID from pairs P where P.id = V.pairid)), (select rtext from methods where id = (select rightMethodID from pairs P where p.id = V.pairid)) from verifiedpairs V where consensus = 1;
```

## Detailed Information

### Table `methods`

Table `methods` includes various information related to methods.
The schema of table `methods` is as follows.

```shell-session
sqlite> .schema methods
CREATE TABLE methods (signature string, name string, rtext blob, ntext blob, size int, branches int, hash blob,path string, start int, end int, repo string, revision string, compilable int, tests int, Target_ESTest blob, Target_ESTest_scaffolding blob, groupID int, id integer primary key autoincrement);
CREATE UNIQUE INDEX sameness on methods (path, start, end, repo, revision);
```

- `signature` represents the text of signature information including return type and parameter types of the method.
- `name` represents the name of the method.
- `rtext` represents the raw text of the method.
- `ntext` represents the normalized text of the method.
- `size` represents the number of program statements included in the method.
- `branches` represents the number of branches included in the method.
- `hash` represents the MD5 hash value of the normalized text of the method.
- `path` represents the path to the file including the method.
- `start` and `end` represent the start/end line of the method in the file.
- `repo` represents the repository including the method.
- `revision` represents the commit id of the repository where the method was extracted to the database.
- `compilable`

