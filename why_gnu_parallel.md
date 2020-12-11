### Why GNU-parallel?

With GNU-parallel one can easily transform workflows to run steps in parallel. Jobs run in parallel, and it's easy to change commands, options and number of parallel jobs, no more for loops. More details at https://www.gnu.org/software/parallel/parallel_tutorial.html

To install GNU-parallel with conda
```
conda install parallel
```

### Example 1
```bash
seq 1 5 | parallel -k "echo 'Welcome {} times"
```
is equal to for loop below. `-k` keeps out put in order. More imporatantly, the `echo` commands were run in parallel, not sequencially. 

```bash
for i in {1..5}
do
   echo "Welcome $i times"
done
```

### Example 2
Create a folder `test` and 5 sample files in it.
```
mkdir test
touch test/sample_{A..E}.txt
ls test/sample_*txt
```
Show files: 
```
test/sample_A.txt  test/sample_B.txt  test/sample_C.txt  test/sample_D.txt  test/sample_E.txt
```

Command to show how to process 5 jobs in parallel. 
```
ls test/sample_*txt | parallel "echo order = {#}, file_paths = {}, base = {.}, fileBase = {/.}, fileName = {/}, fileFolder = {//}"
```
Give out put like below:
```
order = 2, file_paths = test/sample_B.txt, base = test/sample_B, fileBase = sample_B, fileName = sample_B.txt, fileFolder = test
order = 1, file_paths = test/sample_A.txt, base = test/sample_A, fileBase = sample_A, fileName = sample_A.txt, fileFolder = test
order = 4, file_paths = test/sample_D.txt, base = test/sample_D, fileBase = sample_D, fileName = sample_D.txt, fileFolder = test
order = 3, file_paths = test/sample_C.txt, base = test/sample_C, fileBase = sample_C, fileName = sample_C.txt, fileFolder = test
order = 5, file_paths = test/sample_E.txt, base = test/sample_E, fileBase = sample_E, fileName = sample_E.txt, fileFolder = test
```
