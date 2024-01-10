Class 3 – For loops and computing on multiple files
===============================

Goal
----

- In this module, We will apply for loops to perform same repetitive tasks on different files

Using for loops to perform same actions on different files
----------------------------------------------------------

So, using the wildcard in the command above did not have the desired action, as it told us the total number of lines in all files combined, instead of in each individual one. What we need is a way to execute our wc command on each file, but using a single command. You might say to yourself that it isn't a big deal to just run the command three times (once for each file), but what if you had 10 sequences? Or 100 sequences? It becomes inefficient and error prone, so we want to avoid that. What we are going to do instead is use a for loop, to loop over each file and run our command. For loops are not something unique to Unix, and are a fundamental tool for most programming languages for executing repetitive tasks in a streamlined way.


To get a sense for how for loops look in Unix and how they work, try the below example of for loop, that loops over a bunch of numbers and prints out each value until the list is exhausted.

```
for i in 1 2 3 4 5; do echo "Looping ... number $i"; done
```

The above for loop has the following key pieces:

1. for - All for loops start with the word 'for', to indicate the loop command is starting
2. Loop variable - In our case the loop variable is 'i'. We call it i here, but it could be called anything. The loop variable is what is going to change each time through the loop, and allow us to perform a single command in slightly different ways (e.g. run our grep/wc pipe on different files). 
3. Loop list - In our case the loop list is '1 2 3 4 5'. The loop list is the set of things you want to apply your command to. 
4. do - All for loops have the word 'do', which indicates that you are about to enter the command that will be repeated each time through the loop
5. Loop command(s) - After do comes the command that you want to run. A few things to note: i) this command should always include the loop variable, as that is what is going to change each time the loop statement is executed and ii) when you use the loop variable you need to preface it with a $ (e.g. $i)
6. done - All for loops have the word 'done', which indicates that the loop is ending. The commands between 'do' and 'done' get executed each time through the loop

So, what actually happens when we run the above for loop? Well, the following occurs:

1. The loop variable i is assigned 1 (the first value in our loop list)
2. echo "Looping ... number 1" is executed
3. The loop variable i is assigned 2 (the second value in our loop list)
4. echo "Looping ... number 2" is executed
5. The loop variable i is assigned 3 (the third value in our loop list)
6. echo "Looping ... number 3" is executed
7. The loop variable i is assigned 4 (the fourth value in our loop list)
8. echo "Looping ... number 4" is executed
9. The loop variable i is assigned 5 (the fifth value in our loop list)
10. echo "Looping ... number 5" is executed

OK - now let's create a for loop to count the number of sequences in each of our files!

```
for filename in E_coli.fna Kleb_pneu.fna Acinetobacter_baumannii.fna;
do
grep ">" $filename | wc -l
done
```
It works! One thing that would make it better is if the name of the file was printed out along with the number of sequences. Can you add a line to the for loop to accomplish this ?

Hint - look at how we printed something to the screen in our counting loop.

<details>
  <summary>Solution</summary>

```
for filename in E_coli.fna Kleb_pneu.fna Acinetobacter_baumannii.fna;
do
echo $filename
grep ">" $filename | wc -l
done
```
</details>


Turning our fasta counter code into a shell script
--------------------------------------------------

Above we applied our for loop to count the number of sequences in a fasta file to two different directories by copying over the code and putting it in a new file. This worked, but is actually bad practice for a few reasons:
1) If you come up with a way to improve your loop (e.g. provide some more informative print outs), you'd have to change it in all the copies you made
2) Even worse, if you find a bug in your code, you have to fix the bug in all copies, which can be tedious and error prone

To avoid copying over our code, we can instead store the code in a file called a shell script, that we can then call everytime we want to run the code! 

Shell scripts are just the Unix version of a computer program, and are comprised of sets of Unix commands. 

So, first let's put our for loop into a file called 'fasta_counter.sh'. 

Use nano to open the file.

```
nano fasta_counter.sh
```

Add the below for loop in fasta_counter.sh shell script.

```
for fasta_file in *.fna
do

    #PRINT THE NAME OF THE FILE
    echo $fasta_file;
    #Count the number of sequences in fasta_file
        grep '>' $fasta_file | wc -l;

done
```

Now, this is nice, but as it stands the shell script will only count the number of sequences in the directory in which it is stored. What would be even better is if we could apply it to count the number of sequences in any directory! To do this, we can give our script a command line argument. Command line arguments are additional information that you provide a shell script, which the shell script can then use to run in a slightly different manner. In bash, command line arguments go into special variables with numeric names (e.g. $1 for the first command line arugment, $2 for the second, etc.) 


Lets take an example to understand what those parameters stands for:


```
./some_program.sh Argument1 Argument2 Argument3
```

In the above command, we provide three command line Arguments that acts like an input to some_program.sh 
These command line argument inputs can then be used to inside the scripts in the form of $0, $1, $2 and so on in different ways to run a command or a tool.

For this script $1 would contain "Argument1" , $2 would contain "Argument2" and so on...

Lets try to incorporate a for loop inside the fasta_counter.sh script that uses the first command line argument - i.e directory name and search for \*.fna files in that directory and run fasta sequence counting command on each of them.

- Open “fasta_counter.sh” in nano.

- Input the for loop to count the number of sequences in a fasta file

- Add $1 in the appropriate place where the directory placeholder is needed.

- Run this script on the more_fasta directory and verify that you get the correct results. 

- Basic usage of the script will be:

```
./fasta_counter.sh more_fasta
```

You might get an error saying Permission denied. What's the reason? How do we check the permission and set appropriate permission for this file?

<details>
  <summary>Solution</summary>

```
 
ls -l fasta_counter.sh
    
chmod u=rwx fasta_counter.sh 

```
</details>