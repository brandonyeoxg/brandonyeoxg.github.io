+++
draft = true
date = 2021-12-12T20:27:55+08:00
title = "GOtchas Part 1"
description = "Covering gotchas in GO"
tags = ["golang", "tidbits", "gotchas"]
categories = ["golang"]
series = ["golang", "gotchas"]
+++
> In order for me to build the habit of posting weekly content, hopefully every sunday, I would need to reduce the scope of what I will be sharing. Hopefully this reduction in scope will make me motivated habitually of post useful software engineering content.

One of the common gotchas in GO relates to the use of slices. In gist a slice is a dynamically sized array. Within the concrete implementation of a slice are 3 fields:
1. Pointer to first element of  the underlying array holding the values
2. The length of the slice
3. The capacity of the slice

As mentioned since a slice is dynamically sized, we can append to a slice using the `append([]T s, ...T)` in built function.
> NOTE: In this artical T represents the type

Furthermore we can initialise a slice using the `make([]T s, len, cap)` in built function.

A gotcha however arises from these 2 functions, let's take a deeper look.

Consider an example of adding users to a list, we can do the following:
```golang
users := []string{"alice", "bob", "charlie", "dan", "erica"}
```

Now consider that we would like filter out these users maybe in a whitelist:
```golang
users := []string{"alice", "bob", "charlie", "dan", "erica"}
// Whitelist users: "alice", "bob" and "charlie".

// Approach 1 (in place)
for i := 0; i < len(users); {
  if !isWhitelisted(users[i]) {
    users = append(users[:i], users[i+1:]...)
    continue
  }
  i++
}

// Approach 2 immutable
wUsers := make([]string)
// or wUsers := []string{}


for _, u := range users {
  if !isWhitelisted(u) {
    continue
  }
  wUsers = append(wUsers, u)
}

```
Both approaches are well and good (I prefer approach 2), but can we do better? What if we preallocate memory so that our slice does not have to double in size whenever its underlying array size cannot hold the incoming value. We can make use of `make([]T s, len, cap)` touched briefly above for this:
```golang
users := []string{"alice", "bob", "charlie", "dan", "erica"}
// Whitelist users: "alice", "bob" and "charlie".

wUsers := make([]string, len(users))

for _, u := range users {
  if !isWhitelisted(u) {
    continue
  }
  wUsers := append(users, u)
}
```
There we're done right? We have our final whitelisted users? No, no we don't. The output of the above code will be:
```golang
users := []string{"alice", "bob", "charlie", "dan", "erica"}
// Whitelist users: "alice", "bob" and "charlie".

wUsers := make([]string, len(users))

for _, u := range users {
  if !isWhitelisted(u) {
    continue
  }
  wUsers := append(users, u)
}

fmt.Println("wUsers", wUsers, "len", len(wUsers), "cap", cap(wUsers)) // wUsers [   alice bob charlie] len 6 cap 6
```
So what actually happened? Simply put it was the confusion between what is a length and capacity in a slice. When we did:
```golang
wUsers := make([]string, len(users))
```
What we did was to assign len and cap of the slice to `len(users)`. The length of the slice is what `append([]T s, ...T)` will use to figure out which index the new value should be slotted into the underlying array of the slice. So when we called append in the first iteration, we actually had to increase the size of the underlying array and placed the first element of `users` into the 4th index of `wUsers`. This carried on for the rest of the whitelisted users to be added.

The solution would be as follows:
```golang
users := []string{"alice", "bob", "charlie", "dan", "erica"}
// Whitelist users: "alice", "bob" and "charlie".

wUsers := make([]string, 0, len(users)) // Here we set the len == 0 and cap == len(users)

for _, u := range users {
  if !isWhitelisted(u) {
    continue
  }
  wUsers := append(wUsers, u)
}

fmt.Println("wUsers", wUsers, "len", len(wUsers), "cap", cap(wUsers)) // wUsers [a b c] len 3 cap 3
```

Fun fact, I made this same exact noob mistake which introduced a bug in staging and suffice to say I learnt from this embarrassing mistake.

I hope this was as informative to you as it was me. Thank you for making this far!
