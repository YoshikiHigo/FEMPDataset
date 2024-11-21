# FEMPDataset

FEMPDataset is a dataset of functionally equivalent (in short, FE) method pairs.
FEMPDataset includes 1,342 FE method pairs that have been validated by three programmers.


## Quick start

First of all, you can download the dataset from the following URL:
https://www.dropbox.com/s/7rcg4mso1k755nh/ijadataset.db?dl=0

The size of this dataset is very large, approximately 2.67 GB. For this reason, this file is not on GitHub, but on Dropbox.

Then, please make sure that SQLite is installed in your environment.
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

The three reviewers are master's students, all of whom have programming experience using Java.
The three reviewers had the following working time to make individual judgements.
- `Reviewer-A`: 44 hours 48 minutes,
- `Reviewer-B`: 33 hours 20 minutes,
- `Reviewer-C`: 43 hours 25 minutes.

They also spent a total of 9 hours and 28 minutes in discussion to reach a consensus on the method pairs that differed in their individual judgements.

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
- `repo` is not used in this dataset.
- `revision` is not used in this dataset.
- `compilable` is set to `1` if the method is compilable. If not, it becomes `0`. If the method is out of scope for investigating functional equivalence, it becomes `-1`.
- `Target_ESTest` is the set of test cases that Evosuite generated for the method.
- `Target_ESTest_scaffolding` is the parent class of `Target_ESTest`. This source code is also generated by Evosuite.
- `groupID` is not used in this dataset.
- `id` represents the unique identifier of the method.

Of the above items, `repo`, `revision`, and `groupID` are not used in this dataset.
So, all users of this dataset can ignore values in those items.


### Table `pairs`

Table `pairs` includes a list of method pairs that are candidates of FE method pairs.
The schema of table `pairs` is as follows.

```shell-session
sqlite> .schema pairs
CREATE TABLE pairs (leftMethodID int, rightMethodID int, id integer primary key autoincrement);
```

- `leftMethodID` and `rightMethodID` represent the identifiers of the two methods that form the pair. `leftMethodID/rightMethodID` are common to `id` in table `methods`.
- `id` is the unique identifier of this pair.

For example, you can obtain the raw code of method pairs that are candidates of functionally equivalent ones with the following command.

```shell-session
sqlite> select (select M1.rtext from methods M1 where M1.id = p.leftMethodID), (select M2.rtext from methods M2 where M2.id = p.rightMethodID) from pairs P;
```

Herein, each candidate of FE method pairs satisfies all the following conditions.
- Five or more test cases have been generated from each method included in the pair.
- Let `Method-A` and `Method-B` be the two methods that forms the pair. `Method-A` passes all test cases generated from `Method-B` and `Method-B` passes all test cases generated from `Method-A`.


### How to extract method pairs in `pairs` for manual checking

Table `pairs` includes 13,710 candidates of FE method pairs.
However, it is not practical to manually check such a large number of candidates one by one.
Therefore, some of them were extracted and subjected to manual verification in this dataset.
The extraction was performed with the following procedure.

1. Initialize `selectedPairs` and `selectedMethods` to be empty.
2. List the method pairs in Table `pairs` in the ascending order by `id`.
3. For each method pair, if neither method of the method pair is included in `selectedMethods`, add the method pair to `selectedPairs` and add the two methods to `selectedMethods`. If either of the method pair is already included in `selectedMethods`, do do nothing for the method pair.

The method pairs included in `selectedPairs` after the above process are the method pairs to be verified manually.
The above process resulted in the extraction of 2,195 method pairs.

## Use in your research

If you are using FEMPDataset in your research, please cite the following paper:

Yoshiki Higo, "Dataset of Functionally Equivalent Java Methods and Its Application to Evaluating Clone Detection Tools", IEICE Transactions on Information and Systems, Vol.E107-D, No.6, pp.751--760, June 2024. [[available online](https://doi.org/10.1587/transinf.2023EDP7268)]
```
@article{YoshikiHIGO.2023EDP7268,
  title={Dataset of Functionally Equivalent Java Methods and Its Application to Evaluating Clone Detection Tools},
  author={Yoshiki HIGO},
  journal={IEICE Transactions on Information and Systems},
  volume={E107.D},
  number={6},
  pages={751--760},
  year={2024},
  doi={10.1587/transinf.2023EDP7268}
}
```

## Other dataset
[[PyFuncEquivDataset](https://github.com/Scepter4Qing/PyFuncEquivDataset){:target="_blank"}]: functionally equivalent dataset on Python code
