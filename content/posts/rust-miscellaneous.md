+++
title = 'Rust Misc'
date = 2023-12-09T23:40:15+08:00
summary = "Rust miscellaneous"
tags = ["Rust", "Programming"]
+++

Here I record my way to meet the need, if I found better way, I'll update it.

## Apply Judge Keywords List for a Keyword List

``` rust
#[test]
fn test_judge_keyword() {
    let judge_keywords_list = [&["A", "B"], &["C", "D"]];

    let keywords = ["H", "C", "A"];

    let result = judge_keywords_list
        .iter()
        .map(|judge_keywords| {
            let count = judge_keywords
                .iter()
                .fold(0, |contains_count, judge_keyword| {
                    if keywords.contains(judge_keyword) {
                        contains_count + 1
                    } else {
                        contains_count
                    }
                });
            count == 1
        })
        .all(|x| x);

    assert!(result)
}
```

## Interior mutable

https://doc.rust-lang.org/std/cell/index.html

## Debug

debug with gdb, you should type `break <bin-name>::main` first.

[![asciicast](https://asciinema.org/a/wmTg5X6cAnoPhB0GmVup3Hwab.svg)](https://asciinema.org/a/wmTg5X6cAnoPhB0GmVup3Hwab)
