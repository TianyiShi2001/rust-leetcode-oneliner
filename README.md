# rust-leetcode-oneliner

My one-liner solutions to leetcode problems by only chaining methods provided by the `Iterator` trait.

These solutions aren't necessarily efficient, but they are fun! They also serve as good demonstrations of the use of rust iterators.

## TOC
- [TOC](#toc)
- [Oneliners](#oneliners)
  - [66. Plus One](#66-plus-one)
  - [*147. Insertion Sort List](#147-insertion-sort-list)
  - [271. Contains Duplicate](#271-contains-duplicate)
  - [884. Uncommon Words from Two Sentences](#884-uncommon-words-from-two-sentences)
- [Implementing `Iterator` (not really one-liner)](#implementing-iterator-not-really-one-liner)
  - [17. Letter Combinations of a Phone Number](#17-letter-combinations-of-a-phone-number)
  - [885. Spiral Matrix III](#885-spiral-matrix-iii)

## Oneliners

### [66. Plus One](https://leetcode.com/problems/plus-one/)

```rust
pub fn plus_one(digits: Vec<i32>) -> Vec<i32> {
    digits
        .iter()
        .rev()
        .skip(1)
        .chain(std::iter::once(&0i32))
        .fold(
            (
                vec![(digits[digits.len() - 1] + 1) % 10],
                digits[digits.len() - 1] == 9,
            ),
            |(mut acc, carry), curr| {
                let mut curr = if carry { *curr + 1 } else { *curr };
                let carry = if curr == 10 {
                    curr = 0;
                    true
                } else {
                    false
                };
                acc.push(curr);
                (acc, carry)
            },
        )
        .0
        .into_iter()
        .rev()
        .skip_while(|&x| x == 0)
        .collect()
}
```

### *[147. Insertion Sort List](https://leetcode.com/problems/insertion-sort-list/submissions/)

```rust
pub fn insertion_sort_list(head: Option<Box<ListNode>>) -> Option<Box<ListNode>> {
    head.map(|mut head| {
        let mut curr = head.as_mut() as *mut ListNode;
        unsafe {
            while let Some(curr_next) = (*curr).next.as_mut() {
                if curr_next.val < head.val {
                    std::mem::swap(&mut curr_next.val, &mut head.val);
                }
                curr = curr_next.as_mut() as *mut ListNode;
            }
        }
        Box::new(ListNode {
            val: head.val,
            next: Solution::insertion_sort_list(head.next),
        })
    })
}
```

### [271. Contains Duplicate](https://leetcode.com/problems/contains-duplicate/)

```rust
pub fn contains_duplicate(nums: Vec<i32>) -> bool {
    nums.into_iter()
        .fold(std::collections::HashMap::new(), |mut map, num| {
            let count = map.entry(num).or_insert(0usize);
            *count += 1;
            map
        })
        .values()
        .any(|&count| count > 1)
}
```

```rust
pub fn contains_duplicate(nums: Vec<i32>) -> bool {
    nums
        .into_iter()
        .try_fold(std::collections::HashSet::new(), |mut set, num| {
            if set.contains(&num) {
                return None;
            } else {
                set.insert(num);
                Some(set)
            }
        })
        .is_none()
```

### [884. Uncommon Words from Two Sentences](https://leetcode.com/problems/uncommon-words-from-two-sentences/)

```rust
pub fn uncommon_from_sentences(a: String, b: String) -> Vec<String> {
    a.split_whitespace()
        .chain(b.split_whitespace())
        .fold(std::collections::HashMap::new(), |mut m, w| {
            let c = m.entry(w).or_insert(0usize);
            *c += 1;
            m
        })
        .iter()
        .fold(Vec::new(), |mut v, (&s, &f)| {
            if f == 1 {
                v.push(s.to_owned())
            }
            v
        })
}
```

## Implementing `Iterator` (not really one-liner)

### [17. Letter Combinations of a Phone Number](https://leetcode.com/problems/letter-combinations-of-a-phone-number/)

```rust
pub fn letter_combinations(digits: String) -> Vec<String> {
    Generator::new(digits).collect()
}

struct Generator {
    letters: Vec<Vec<char>>,
    total_lengths: Vec<usize>,
    states: Vec<usize>,
    finished: bool,
}

impl Generator {
    fn new(inp: String) -> Self {
        let letters: Vec<Vec<char>> = inp
            .chars()
            .map(|d| match d {
                '2' => vec!['a', 'b', 'c'],
                '3' => vec!['d', 'e', 'f'],
                '4' => vec!['g', 'h', 'i'],
                '5' => vec!['j', 'k', 'l'],
                '6' => vec!['m', 'n', 'o'],
                '7' => vec!['p', 'q', 'r', 's'],
                '8' => vec!['t', 'u', 'v'],
                '9' => vec!['w', 'x', 'y', 'z'],
                _ => unreachable!(),
            })
            .collect();
        let total_lengths = letters.iter().map(|x| x.len() - 1).collect();
        Self {
            letters,
            total_lengths,
            states: vec![0; inp.len()],
            finished: inp.len() == 0,
        }
    }
}

impl Iterator for Generator {
    type Item = String;
    fn next(&mut self) -> Option<Self::Item> {
        if self.finished {
            None
        } else {
            let res = self.letters.iter().zip(self.states.iter()).fold(
                String::new(),
                |mut s, (chars, &idx)| {
                    s.push(chars[idx]);
                    s
                },
            );
            if let Err(pos) = self
                .states
                .iter_mut()
                .zip(self.total_lengths.iter())
                .try_fold(0usize, |pos, (idx, total)| {
                    if *idx < *total {
                        *idx += 1;
                        Err(pos)
                    } else {
                        Ok(pos + 1)
                    }
                })
            {
                self.states.iter_mut().take(pos).for_each(|x| *x = 0);
            } else {
                self.finished = true;
            }
            Some(res)
        }
    }
}
```

### [885. Spiral Matrix III](https://leetcode.com/problems/spiral-matrix-iii)

```rust
pub fn spiral_matrix_iii(r: i32, c: i32, r0: i32, c0: i32) -> Vec<Vec<i32>> {
    Spiral::new(r0, c0, Direction::Right, true)
        .try_fold(Vec::new(), |mut v, (i, j)| {
            if (i >= 0 && i < r) && (j >= 0 && j < c) {
                v.push(vec![i, j]);
            }
            if v.len() as i32 == r * c {
                Err(v)
            } else {
                Ok(v)
            }
        })
        .unwrap_or_else(|v| v)
}

pub enum Direction {
    Up,
    Right,
    Down,
    Left,
}

pub struct Spiral {
    pub r: i32,
    pub c: i32,
    direction: Direction,
    clockwise: bool,
    n: usize,
    m: usize,
}

impl Spiral {
    pub fn new(r: i32, c: i32, direction: Direction, clockwise: bool) -> Self {
        Self {
            r,
            c,
            direction,
            clockwise,
            n: 1,
            m: 2,
        }
    }
    fn step(&mut self) {
        match self.direction {
            Direction::Down => {
                self.r += 1;
            }
            Direction::Right => {
                self.c += 1;
            }
            Direction::Up => {
                self.r -= 1;
            }
            Direction::Left => {
                self.c -= 1;
            }
        }
    }
    fn turn(&mut self) {
        self.direction = match self.direction {
            Direction::Down => {
                if self.clockwise {
                    Direction::Left
                } else {
                    Direction::Right
                }
            }
            Direction::Right => {
                if self.clockwise {
                    Direction::Down
                } else {
                    Direction::Up
                }
            }
            Direction::Up => {
                if self.clockwise {
                    Direction::Right
                } else {
                    Direction::Left
                }
            }
            Direction::Left => {
                if self.clockwise {
                    Direction::Up
                } else {
                    Direction::Down
                }
            }
        };
    }
}

impl Iterator for Spiral {
    type Item = (i32, i32);
    fn next(&mut self) -> Option<Self::Item> {
        let last_coords = (self.r, self.c);
        self.step();
        self.n -= 1;
        if self.n == 0 {
            self.m += 1;
            self.n = self.m / 2;
            self.turn();
        }
        Some(last_coords)
    }
}
```
