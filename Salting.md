Attempts to explain the need for salting passwords to an inexperienced cryptographer have been in my experience, futile. This is especially so when instead of providing a real world example, first an elaborate technical description is said, as is common online. I thought I would change this. This is how I explain salting to coworkers:

## Life before salt

Let's say we have 5 users


| row | Username | Password (Not stored!) | Hash                                                             |
| --- | -------- | ---------------------- | ---------------------------------------------------------------- |
| 1   | bob      | orange                 | 1b4c9133da73a711322404314402765ab0d23fd362a167d6f0c65bb215113d94 |
| 2   | lisa     | dungeon                | 2a79be6a5deb17eb3973b3e1872623682287731df936d313f7c8b0e4a336e958 |
| 3   | alex     | flubber                | 29f006c8fea00ee58303e1d660279900967e7c86c31ae51f8f7c148f32f7f8f1 |
| 4   | joe      | pineapple              | b0fef621727ff82a7d334d9f1f047dc662ed0e27e05aa8fd1aefd19b0fff312c |
| 5   | sarah    | banana                 | b493d48364afe44d11c0165cf470a4164d1e2609911ef998be868d46ade3de4e |

Our attacker has already generated a hash table, this is his table

| row | Password   | Hash                                                             |
| --- | ---------- | ---------------------------------------------------------------- |
| 1   | lemon      | f464d7d71c06e47a535ce441aa202aa717cddeab902a45b0c283aac7a9a090d7 |
| 2   | orange     | 1b4c9133da73a711322404314402765ab0d23fd362a167d6f0c65bb215113d94 |
| 3   | banana     | b493d48364afe44d11c0165cf470a4164d1e2609911ef998be868d46ade3de4e |
| 4   | strawberry | 5e737f891db1175442a39fde73e51d781a545506d71c95477a6deb5988bd7f9a |
| 5   | pineapple  | b0fef621727ff82a7d334d9f1f047dc662ed0e27e05aa8fd1aefd19b0fff312c |

She gains access to our database. Let's assume the following:

- Comparing two strings takes 0.1 seconds
- Generating a hash takes 0.5 seconds

Given this, she launches an attack.

<details>
<summary>attack</summary>
- userDB row 1 matches hashDB row 2 (0.2 seconds)
- userDB 2 matches hashDB none (0.5 seconds)
- u3 = hnone (0.5 seconds)
- u4 = h5 (0.5 seconds)
- u5 = h3 (0.3 seconds)
</details>

In her attack, for each user, she tries each hash until one matches. As soon as a hash works, she moves onto the next user. This means that for passwords she does not have hashes of already she spends 0.5 seconds because her hash table has 5 entries and she needs to make sure the hash is not in that table. Sometimes, she finds a mashing hash fast, say in 0.2 seconds.

All in all, her attack took 2 seconds, and she walked away with 3 accounts.

## Life after salt

Let's modify our database to *salt* our passwords