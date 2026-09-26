# Assignment #2 — Report

**Name: Omckar Savlani**
**Student ID: 801497440**
**Email: osavlani@charlotte.edu**

---

## Design

Which design did you choose (A, B, or your own)? Explain in your own words:

- What your **Mapper** emits as key and value, and why that is the right thing to emit.
- What your **Reducer** receives for one key, what it does with it, and where the Jaccard
  similarity is computed.
- What you had to set in the **Driver** beyond what L4's `Controller` set, and why.

Design A.

Mapper — emits (documentID, "word1 word2 word3 ..."). The key is the document ID, and Design A needs every document's contents grouped by its own ID so the reducer can hold as one unit. The value is the cleaned, deduplicated set of words (lowercased, punctuation stripped) since Jaccard only cares about which words appear, not how many times — turning duplicates into repeats would just waste space and complicate the set math later for no benefit.

Reducer — for one key (one document), it gets an iterable of values, and converts words into a set and stores it in a map keyed by document ID. It fires once per document, no comparisons, so at any given call it's only ever seen one document, and Jaccard needs two. The comparison happens in cleanup(), which Hadoop calls once after every single reduce() call has finished.

Driver — beyond what L4's Controller set up, the one thing that really matters here is job.setNumReduceTasks(1). The whole setup only works if every document ends up on the same reducer, because cleanup() can only compare documents it has physically stored, and if the job split across two reducers, each one would only ever see half the documents and never find their overlap.

---

## How I ran it

The commands you used, in the order you used them. If you deviated from the steps in the
README, say where and why.

```bash
docker --version
java -version
mvn -version
# To check compatibility, verify setup

docker compose -f docker-compose.codespaces.yml up -d
# To compose codespaces.yml specifically

docker ps
# to check all nodes are active

mvn clean package

docker cp target/DocumentSimilarity-0.0.1-SNAPSHOT.jar resourcemanager:/tmp/
docker cp shared-folder/input/data/small_dataset.txt resourcemanager:/tmp/
docker cp shared-folder/input/data/dataset.txt resourcemanager:/tmp/
# copying files to container /tmp

docker exec -it resourcemanager bash
cd /tmp

hadoop fs -mkdir -p /input/data
hadoop fs -put ./small_dataset.txt /input/data
hadoop fs -put ./dataset.txt /input/data
hadoop fs -ls /input/data

docker network ls
# name nodes were failing, retried with H4 cosedspaces.yml file

hadoop jar /tmp/DocumentSimilarity-0.0.1-SNAPSHOT.jar \
  com.example.controller.DocumentSimilarityDriver /input/data/small_dataset.txt /output/small_dataset

hadoop fs -cat /output/small_dataset/* :

# bash-4.2$ hadoop fs -cat /output/small_dataset/*
#Document1, Document2 Similarity: 0.18
#Document1, Document3 Similarity: 0.20
#Document2, Document3 Similarity: 0.10

#To check results of test dataset, they match with provided task, test passed.
hadoop jar /tmp/DocumentSimilarity-0.0.1-SNAPSHOT.jar \
  com.example.controller.DocumentSimilarityDriver /input/data/dataset.txt /output/dataset

# ran test on real dataset

hadoop fs -cat /output/dataset/*
hdfs dfs -get /output /tmp/
exit

#copied test ouputs to tmp

docker cp resourcemanager:/tmp/output/. shared-folder/output/
# copied output to repo

docker compose -f docker-compose.codespaces.yml down
# shut down the container
```

---

## Output

### `small_dataset.txt` (3 lines)

```
Document1, Document2 Similarity: 0.18
Document1, Document3 Similarity: 0.20
Document2, Document3 Similarity: 0.10
```

### `dataset.txt` (66 lines)

```
Doc01, Doc02 Similarity: 0.16
Doc01, Doc03 Similarity: 0.13
Doc01, Doc04 Similarity: 0.07
Doc01, Doc05 Similarity: 0.10
Doc01, Doc06 Similarity: 0.09
Doc01, Doc07 Similarity: 0.11
Doc01, Doc08 Similarity: 0.10
Doc01, Doc09 Similarity: 0.11
Doc01, Doc10 Similarity: 0.09
Doc01, Doc11 Similarity: 0.07
Doc01, Doc12 Similarity: 0.19
Doc02, Doc03 Similarity: 0.20
Doc02, Doc04 Similarity: 0.13
Doc02, Doc05 Similarity: 0.10
Doc02, Doc06 Similarity: 0.09
Doc02, Doc07 Similarity: 0.06
Doc02, Doc08 Similarity: 0.09
Doc02, Doc09 Similarity: 0.05
Doc02, Doc10 Similarity: 0.10
Doc02, Doc11 Similarity: 0.06
Doc02, Doc12 Similarity: 0.14
Doc03, Doc04 Similarity: 0.17
Doc03, Doc05 Similarity: 0.11
Doc03, Doc06 Similarity: 0.08
Doc03, Doc07 Similarity: 0.16
Doc03, Doc08 Similarity: 0.11
Doc03, Doc09 Similarity: 0.07
Doc03, Doc10 Similarity: 0.10
Doc03, Doc11 Similarity: 0.12
Doc03, Doc12 Similarity: 0.11
Doc04, Doc05 Similarity: 0.09
Doc04, Doc06 Similarity: 0.11
Doc04, Doc07 Similarity: 0.18
Doc04, Doc08 Similarity: 0.09
Doc04, Doc09 Similarity: 0.08
Doc04, Doc10 Similarity: 0.10
Doc04, Doc11 Similarity: 0.09
Doc04, Doc12 Similarity: 0.09
Doc05, Doc06 Similarity: 0.20
Doc05, Doc07 Similarity: 0.14
Doc05, Doc08 Similarity: 0.15
Doc05, Doc09 Similarity: 0.07
Doc05, Doc10 Similarity: 0.13
Doc05, Doc11 Similarity: 0.14
Doc05, Doc12 Similarity: 0.11
Doc06, Doc07 Similarity: 0.17
Doc06, Doc08 Similarity: 0.15
Doc06, Doc09 Similarity: 0.08
Doc06, Doc10 Similarity: 0.10
Doc06, Doc11 Similarity: 0.12
Doc06, Doc12 Similarity: 0.13
Doc07, Doc08 Similarity: 0.15
Doc07, Doc09 Similarity: 0.07
Doc07, Doc10 Similarity: 0.08
Doc07, Doc11 Similarity: 0.12
Doc07, Doc12 Similarity: 0.11
Doc08, Doc09 Similarity: 0.19
Doc08, Doc10 Similarity: 0.13
Doc08, Doc11 Similarity: 0.22
Doc08, Doc12 Similarity: 0.12
Doc09, Doc10 Similarity: 0.13
Doc09, Doc11 Similarity: 0.12
Doc09, Doc12 Similarity: 0.13
Doc10, Doc11 Similarity: 0.12
Doc10, Doc12 Similarity: 0.12
Doc11, Doc12 Similarity: 0.11
```

---

## Analysis

Look at the results for `dataset.txt`.

- Which pairs are the most similar, and which the least?
- Do the most similar pairs make sense given what the documents are about?
- The values are all fairly low and close together. Why? What one change to the tokenization
  rules would make the numbers more meaningful?

Most similar: Doc08–Doc11 (0.22), then Doc02–Doc03 and Doc05–Doc06 (both 0.20).
Least similar: Doc02–Doc09 (0.05), Doc02–Doc07 (0.06), Doc02–Doc11 (0.06).

Doc02 shows up repeatedly at the bottom — it shares less common words with most other docs.

The tokenization matches identical strings between both docs. It is a partially reliable metric for simiarity. We can also add filters like matching string contents while also matching indexes of those strings or tokenization of the strings in comparison to the strings that appear nearby to them.

---

## Scalability

**If you used Design A:** it relies on a single reducer that holds every document in memory.
What concretely breaks when the collection has a million documents? Sketch how Design B
avoids the problem.

**If you used Design B:** why did it need more than one pass (or how did you avoid that)?
What is its own bottleneck?

Design A (implemented): it breaks down badly at scale because everything hinges on a single reducer. The documents map holds every document's full word set in memory for the entire job. It also gets zero parallelism: only one reducer task ever runs, so adding more nodes to the cluster does nothing to speed up cleanup().

How Design B avoids it: by keying on word instead of document ID, the "which documents share this word" grouping is done by the shuffle itself, across as many reducers as there are distinct words — so the work is spread across the cluster instead of collapsing onto one machine. The tradeoff is it needs a second pass to turn "shared word counts" into a normalized Jaccard score. So basic sacalability depends on how we can accomoate more reducers which is our bottleneck as their functions have complexity of O(n^2).

---

## Problems and fixes

Anything that went wrong and what resolved it. Paste the actual error message. If nothing
went wrong, say so.

Had to figure out which codespaces.yml file was compatible for A2. Professor maybe mentioned H4 but I remembered H6 which wasn't.

---

## Use of generative AI

If you used a generative AI tool, include the acknowledgment statement from the syllabus and
say specifically what you used it for. If you did not use one, say so.

Used to debug the 3 java functions.