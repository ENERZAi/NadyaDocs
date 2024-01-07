# List

List module provides definition for list, and provides functions that performs frequently used
operations on list.

## Components
### len lst
Returns length of the list

__Example__
```c#
module core.List.List as list
let fruits = ["Apple";"Banana";"Pear"]
let ans = list.len fruits
print(ans)
```
__Program output__
```Bash
3
```

### drop number lst
Drops '_number_' elements from '_lst_'

__Example__
```c#
module core.List.List as list
let lst = [1;2;3;4;5]
let ans = list.drop 3 lst
print(ans)
```

__Program output__
```Bash
Op_Cons (4,Op_Cons (5,Op_Empty ))
```

```c#
type List<a> =
    | ; of a * List<a>
    | []

let rec len lst =
    match lst with
    | []    -> 0
    | h;t   -> 1 + len <| t

let rec drop number lst = match number, lst with
    | 0, l    -> l
    | n, h;t  -> if ((len lst) < n) {
                    except("core.List.List.drop -  Cannot drop " + toStr(n) +
                    " elements from list with length " + toStr(len lst))
                 } else {
                    drop <| (n - 1) <| t
                 }

let rec take number lst = match number, lst with
    | 0, _    -> []
    | n, h;t  -> if ((len lst) < n) {
                    except("list doesn't have enough element")
                 } else {
                    h ; (take <| (n - 1) <| t)
                 }

let reverse target =
    let rec _reverse acc target =
        match target with
        | []    -> acc
        | h;t   -> _reverse (h ; acc) t
    _reverse <| [] <| target

let rec merge left right =
    match left, right with
    | [], r          -> r
    | l, []          -> l
    | lh;lt, rh;rt -> if (lh < rh) {
                            lh ; (merge <| lt <| (rh;rt))
                        } else {
                            rh ; (merge <| (lh;lt) <| rt)
                        }

let rec sort target =
    match target with
    | []    ->  target
    | h;[]  ->  h;[]
    | lst   ->  let hf = (len <| target) >> 1
                let left = take <| hf <| target
                let right = drop <| hf <| target
                (merge <| (sort <| left)) <| (sort <| right)

let rec fold folder state list =
    match list with
    | [] -> state
    | h;t -> fold <| folder <| (folder <| state <| h) t

let rec filter predicate list =
    match list with
     | [] -> []
     | h;t -> if(predicate <| h) { h;(filter predicate t) } else { filter predicate t }

let rec forall predicate list =
    let folder state elem =
        state && (predicate elem)
    fold folder true list

let rec init length initializer =
    match length with
    | 0 -> []
    | n -> initializer;(init (length - 1) initializer)

let concat left right =
    let reversedLeft = reverse left
    let rec concatHelper left right =
        match left with
        | [] -> right
        | h;t -> h;(concatHelper t right)
    concatHelper left right

let item index lst =
    if(len lst <= index){
        except("core.List.List.item - " +
            " Given list has length " + toStr(len lst) +
            " but requested to get item at index " + toStr(index))
    } else {
        let rec itemHelper curIdx lst =
            match curIdx, lst with
            | 0, h;t -> h
            | _, h;t -> itemHelper (curIdx - 1) t
        itemHelper index lst
    }

let toRtTuple lst =
    let folder state elem =
        append(state, !{${elem}})
    fold <| folder <| rtTuple() <| lst
```
{collapsible="true" collapsed-title="Possible implementation"}