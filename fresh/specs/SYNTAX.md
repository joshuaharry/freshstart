Here is all of the syntax required for V1 of the language.

```fresh
fn my_add(x: i32, y: i32) i32 {
  return add(x, y);
}

fn main() i32 {
  x: i32 = add(2, 3);
  y: i32 = sub(3, 4);
  z: i32 = my_add(x, y);
  // This is a comment.
  /* This is also a comment. */
  add(2, sub(3, my_add(2, 2)));
}
```
