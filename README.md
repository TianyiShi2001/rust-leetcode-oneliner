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
