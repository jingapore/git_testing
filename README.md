# Objective
This shows the lower level workings of `git add` and `git commit`.
<br />The steps reference: https://git-scm.com/book/en/v2/Git-Internals-Git-Objects.

# Create Git Objects

|Object Type|File Name |Content                   |Hash-object ID                          |
|------|----------|--------------------------|----------------------------------------|
|blob |Nil       |'hello world 202003221322'|4bbe89d870aa9c7a71a9b98cbf27ba638ba8ec1a|
|blob |test.txt  |'version 202003221333'    |13abe14eca45452ea289011048260ed4c6cc6b5a|
|blob |test.txt  |'version 202003221336'    |be737972b526968af04594872f1f8f1e9afd6ee6|
|tree |Nil |tree with test.txt 'version 202003221333' |704a583efdd0a797252d2e999f6714204cfb42dd|
|tree |Nil|tree with test.txt 'version 202003221333'; and <br />new.txt|198ed19d7243e847fc76a0629526aa702bb0f6bf|
|tree |Nil |tree with test.txt 'version 202003221333'; <br /> new.txt; and <br />bak, which is tree with test.txt 'version 202003221333' |c75519037a994cf4d5a96f69a9651ec2105c3cc2|
|commit |Comment: First commit |first tree above |9ea4b10832e0524d5758775c9859ca16383bd30f|
|commit |Comment: Second commit |second tree above |9c5989925c1cbe95f7148e28ddd49e04b3573f83|
|commit |Comment: Third commit |third tree above |f6e1043b4ff20c0635bb531e54c6a4b06315d6f2|

# Add Git Objects to Index
This is what happens when running `git add`.
<br /> Run the following, to add the first version of `text.txt` (i.e. version 202003221333) to index.
<br />`git update-index --add --cacheinfo 100644 $test_text_version_13333 test.txt`

<br />A tree, itself a git object, is formed from objects (i.e. objects and other trees), in the index.
<br />This is thru the command: `git write-tree`.

<br/>So, what `git add` does is:
- Create a git object (by default a blob), with a hash id; and
- Add the id of this git object to the index.

# Understanding Commit Objects
Commit object typically has:
- Committer (mandatory);
- Comment (mandatory); and
- Parent commit (optional).

To create a commit object, run:
<br />`git commit-tree <tree id> -p <parent commit id>`

# Understanding Refs
Good explanation here: https://stackoverflow.com/questions/26005031/what-does-git-push-do-exactly.
<br />Even after adding commit objects, the `master` branch is still not pointing to the commit object.
<br />Since `master` is in fact a ref, we update the reference to point to the latest commit object.
<br /> `git update-ref refs/heads/master f6e1043b4ff20c0635bb531e54c6a4b06315d6f2`

## Why `master` is not the same as working directory
<br />When we run `git status`, we see following:
<pre><code>On branch master
    Changes not staged for commit:
    (use "git add/rm <file>..." to update what will be committed)
    (use "git checkout -- <file>..." to discard changes in working directory)
        
        deleted:    bak/test.txt
</code></pre>
What happened to `bak/test.txt`?
- Remember that we added it to index using `git write-tree`, as it is a tree object.
- This does not write it into the working directory.
- Rather, it goes into the index, and we subsequently added it to another tree object.
- So, it doesn't appear in the working directory.

# Adding README.md from scratch
We try to add `README.md` to a new commit object, which isn't covered in the git-scm tutorial.
<br />Steps:
- Write `README.md` as a hashed object, with `git hash-object -w README.md`
- Read tree of latest commit into index, specifying `-m`, which merges the tree with the git object representing `README.md`.
- Add the git object to index with:
<pre><code>git update-index --add --cacheinfo 100644\
850ac61118774cf2a2d38f080fb75f006573f1e0 README.md</pre></code>
- Write tree, which creates `5fae5d60d4b756185dacda81be828d324e7cb9f4`.
- Create commit object to tree, and parent to previous commit object.
- Update `refs/heads/master` to point to newest commit tree. It is apparent, when running `git rev-parse HEAD`, that `HEAD` is not on the most recent commit object.

# Bash helpers
Shortcut to refer to object hash sent to `git cat-file`:
<br /> ``git cat-file -p  `find .git/objects -type f | awk -F / '{print $3$4}'` ``

