# rust-leetcode-oneliner

Elegent one-liner solutions to leetcode problems using iterators

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
