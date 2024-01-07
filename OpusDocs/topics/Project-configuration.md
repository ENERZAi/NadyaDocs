# Project configuration

This page shows how opus project can be configured by programmer. We will go build opus program that solves
<a href="https://www.geeksforgeeks.org/longest-increasing-subsequence-dp-3/">Longest increasing sequence</a> problem. 
After reading document, you will understand how opus program is structured.

Project can be configured with _opus-project.yml_ file.

Let's assume we have directory structure as follows. opus-project.yml should be always located in project root folder.

```
root
├── solve
│   ├── solve1.opus   
│   └── solve2.opus
├── main.opus
└── opus_project.yml
```

where solve1.opus and solve2.opus implements two different kinds of algorithm for solving LIS problem
### Code files

__solve1.opus__
```c#
// Solve1.opus
module core.Math.Utils as utils

let mut dp = tensor((1010,), 0i)

// Solves LIS problem with O(N^2) algorithm
let solve n input =
    for (i from 0 to n) {
		dp[(i,)] <- 1
		for (j from 0 to i) {
			if (input[(j,)] < input[(i,)])
				dp[(i,)] <- utils.max <| dp[(i,)] <| (dp[(j,)] + 1)
			else
				_end_
		}
	}
	let mut ans = 0
	for (i from 0 to n) {
		ans <- utils.max <| ans <| dp[(i,)]
	}
	ans
_end_
```
{ collapsible="true" collapsed-title="solve1.opus" }

__solve2.opus__
```c#
//Solve2.opus
module core.Algorithm.Bisect as bisect // Import 'bisect' which implements 'lowerBound'

// Set infinite number
let infinite = 2147483647
// Initialize tensor with infinite value
// This dp tensor will contain last value of LIS with length 'idx'
let mut dp = tensor((1010,), infinite)

// Closure that solves LIS problem with O(n*log(n)) algorithm where 'n' is the length of the sequence
let solve n input =
	for (i from 0 to n) {
        let idx = bisect.lowerBound dp (input[(i,)])
        if (dp[(idx,)] > input[(i,)])
            dp[(idx,)] <- input[(i,)]
            _end_
        else
            _end_
	}
	
    // Find index of the infinite value. This value becomes length of the sequence
    let rec findInf idx =
        if (dp[(idx,)] >= infinite)
            idx
        else
            findInf (idx + 1)
    findInf 0
    _end_
```
{collapsible="true" collapsed-title="solve2.opus"}

#### main.opus {collapsible="true"}
<code-block lang='c#'>
// Let's call basic O(n^2) implementation as 'baseSolve' and fast O(n*log(n)) implementation as 'fastSolve'
module solve.solve1 as baseSolve
module solve.solve2 as fastSolve
// Define the input
let input = tensor((10,), {2,64,125,-23,53,12,654,2,0,1243})
let resultFromBase = baseSolve.solve 10 input
print(resultFromBase) // Should print '5' (2, 64, 125, 654, 1243)
let resultFromFast = fastSolve.solve 10 input
print(resultFromFast) // Should print '5' (2, 64, 125, 654, 1243)
</code-block>

### Configuring the project
Now, we construct the opus-project.yml file, which defines how Opus project is structured
```yaml
# opus-project.yml
project:
	name: Solve LIS
	version: 0.1.0
file:
	- solve:
        - solve1.opus
        - solve2.opus
	- main.opus
```
__project__ section includes basic information about the program.
it includes name of the program, and its version.

__file__ section exactly follows how directory is structured.
This means root directory contains 'solve' directory, and solve directory contains two opus files. (solve1.opus and solve2.opus)
The root directory contains main.opus which can call the program.

One important thing to note here is file structure has import hierarchy. The file 'below' can import file 'above' current file,
but file in 'above' cannot import file 'below'.
For instance, solve2.opus can import solve1, but solve1 cannot import solve2. This rule holds for all files inside _file_ section.

### Executing the program

We can execute the program calling opci from the root directory
```Bash
opci .
```

__Program output__
```Bash
5
5
```
We can see our program correctly calculated length of LIS. 

<seealso>
    <category ref="references">
    <a href="https://www.geeksforgeeks.org/longest-monotonically-increasing-subsequence-size-n-log-n/">Longest increasing sequence (n*log(n))</a>
    </category>
</seealso>