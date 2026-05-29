# Side-by-Side Comparison: Improved vs Original Skill
## Eval-1: Calling Convention Bug

---

## 1. INLINE DEF PATTERN EXPLANATION

### Improved Skill (Lines 32-58)
```markdown
## Step 1: Reproduce and Inspect Using the Inline Def Pattern

Instead of modifying the source file, use the REPL with an inline `def` to capture the argument:

```clojure
;; In your REPL (don't modify the source file yet):
(defn process [& [{:keys [id name email]} :as args]]
  (def captured-args args)     ;; TODO: remove before commit
  (def first-arg (first args)) ;; TODO: remove before commit
  (str id ":" name ":" email))

;; Call the function with the problematic arguments:
(process :id 1 :name "Alice" :email "alice@example.com")
;; => java.lang.NullPointerException

;; Now inspect what was captured at the REPL:
captured-args
;; => (:id 1 :name "Alice" :email "alice@example.com")
;;    ^^^ This is a sequence of 6 elements, not a map!

first-arg
;; => :id
;;    ^^^ This is a keyword, not a map! Destructuring fails.

(type first-arg)
;; => clojure.lang.Keyword
```

**Aha!** The problem is clear: the function receives a *sequence of keywords and values*, but tries to destructure the first element as a map. The destructuring fails, leaving `id`, `name`, and `email` as `nil`.
```

### Original Skill (Lines 55-96)
```markdown
### Using `def` to Capture Arguments

The clj-debug skill emphasizes: **use `def` to pin values you want to inspect**. Let's modify the function temporarily in the REPL to see what it receives:

```clojure
;; Redefine the function to capture its input
(defn process [& args]
  (def raw-args args)  ;; <-- PIN the value
  args)

;; Call it with the original arguments
(process :id 1 :name "Alice" :email "alice@example.com")

;; Now inspect what was passed
raw-args
;; => (:id 1 :name "Alice" :email "alice@example.com")
```

**Key insight:** The function receives a **flat sequence of keywords and values**, not a sequence of maps. The variadic `&` operator collects all arguments into a sequence, but there's no map wrapping.

### Inspect the Destructuring Pattern

Let's see what the original destructuring pattern expects:

```clojure
;; The original pattern: [& [{:keys [id name email]}]]
;; This expects: a sequence of maps

;; Let's create what the pattern EXPECTS:
(def expected-input [{:id 1 :name "Alice" :email "alice@example.com"}])

;; And see what actually arrives:
(def actual-input '(:id 1 :name "Alice" :email "alice@example.com"))

;; The difference:
expected-input
;; => [{:id 1, :name "Alice", :email "alice@example.com"}]

actual-input
;; => (:id 1 :name "Alice" :email "alice@example.com")
```

**The problem:** You're passing **keyword arguments** (like a function call signature), but the function expects **a map as the first element of its variadic sequence**.
```

### Analysis

**Improved Skill:**
- ✅ Shows adding def INSIDE the function definition
- ✅ Clear sequence: write function → evaluate → call → inspect
- ✅ Shows actual captured values AND their types
- ✅ Direct "Aha!" moment from the data
- ✅ More concise (27 lines)
- ✅ Emphasizes TODO comment for cleanup
- ✅ Better matches Borkdude's pattern: modify code, evaluate, run, inspect

**Original Skill:**
- ✅ Shows def at REPL level 
- ✅ Compares expected vs actual side-by-side
- ✅ Educational, shows both perspectives
- ✅ More comprehensive (41 lines)
- ✗ Slightly more scattered (three code blocks instead of one cohesive flow)
- ✓ Clear explanation of the mismatch

**Winner for Clarity:** **Improved** — the single cohesive code block showing add-def → evaluate → call → inspect feels more like a workflow you follow step-by-step.

---

## 2. DISCOVERY NARRATIVE: Hypothesis Formation

### Improved Skill (Lines 32-60)
Immediately after the inline def pattern section, the code output itself tells the story:

```
captured-args
;; => (:id 1 :name "Alice" :email "alice@example.com")
;;    ^^^ This is a sequence of 6 elements, not a map!

first-arg
;; => :id
;;    ^^^ This is a keyword, not a map! Destructuring fails.

(type first-arg)
;; => clojure.lang.Keyword

**Aha!** The problem is clear: the function receives a *sequence of keywords and values*...
```

**Flow:** Inspect output → See the pattern → Recognize the error → State hypothesis

### Original Skill (Lines 79-96)
```markdown
Let's see what the original destructuring pattern expects:

;; The original pattern: [& [{:keys [id name email]}]]
;; This expects: a sequence of maps

;; Let's create what the pattern EXPECTS:
(def expected-input [{:id 1 :name "Alice" :email "alice@example.com"}])

;; And see what actually arrives:
(def actual-input '(:id 1 :name "Alice" :email "alice@example.com"))

;; The difference:
expected-input
;; => [{:id 1, :name "Alice", :email "alice@example.com"}]

actual-input
;; => (:id 1 :name "Alice" :email "alice@example.com")

**The problem:** You're passing **keyword arguments** (like a function call signature), 
but the function expects **a map as the first element of its variadic sequence**.
```

**Flow:** Expected vs Actual comparison → Recognize mismatch → State hypothesis

### Analysis

**Improved Skill:**
- ✅ Hypothesis emerges naturally from the data captured
- ✅ Annotations on the output guide thinking
- ✅ "Aha!" moment feels earned from inspection
- ✅ More like real debugging: "I ran this, I see this pattern"
- ✅ Shorter, more direct

**Original Skill:**
- ✅ Explicitly compares what's expected vs actual
- ✅ Educational, teaches about destructuring patterns
- ✅ More thorough explanation
- ✗ Requires understanding the pattern FIRST before seeing the bug
- ✗ Less like the discovery moment

**Winner for Discovery Narrative:** **Improved** — The hypothesis emerges from *what you see*, not from understanding the pattern in advance.

---

## 3. FIX TESTING BEFORE COMMIT

### Improved Skill (Lines 62-137)

```markdown
## Step 2: Test Both Fix Approaches

You have two options. Let's test each in the REPL before committing.

### Option A: Fix the Caller (Recommended for Most Cases)

Pass a map instead of keyword arguments:

```clojure
;; In REPL:
(defn process [& [{:keys [id name email]}]]
  (str id ":" name ":" email))

;; Test with a map:
(process {:id 1 :name "Alice" :email "alice@example.com"})
;; => "1:Alice:alice@example.com"  ✓ Works!
```

This is the cleanest fix because the function signature is designed to accept a single map. Update your caller:

```clojure
;; Call like this:
(process {:id 1 :name "Alice" :email "alice@example.com"})
```

### Option B: Fix the Function Signature (If You Want Keyword Args)

If you prefer to call with keyword arguments, rewrite the function:

```clojure
;; In REPL - Test the fix:
(defn process [& args]
  (def captured-args args)  ;; Inspect to verify
  (let [{:keys [id name email]} (apply hash-map args)]
    (str id ":" name ":" email)))

(process :id 1 :name "Alice" :email "alice@example.com")
;; => "1:Alice:alice@example.com"  ✓ Works!

;; Verify what was captured:
captured-args
;; => (:id 1 :name "Alice" :email "alice@example.com")  ✓ Correct
```

Then update your source file to this version.

## Step 3: Verify Your Choice and Commit

Once you've confirmed your fix works in the REPL:

1. **Remove all `def` lines** (they were only for debugging)
2. **Update the source file** with your chosen fix
3. **Run your test suite** to confirm it solves the original problem
```

**Flow:**
1. Test Option A in REPL (show it works)
2. Test Option B in REPL (show it works)
3. THEN talk about updating source
4. Clear checklist for cleanup

### Original Skill (Lines 101-147)

```markdown
## Step 3: Test the Fix in the REPL (Before Modifying Source)

**Goal:** Verify your hypothesis by testing different calling conventions.

### Fix Option 1: Accept a Map, Pass a Map

```clojure
;; Redefine to accept a map
(defn process [& [{:keys [id name email]}]]
  (str id ":" name ":" email))

;; Call it correctly: wrap arguments in a map
(process {:id 1 :name "Alice" :email "alice@example.com"})
;; => "1:Alice:alice@example.com"  ✓ SUCCESS!
```

**Why this works:** The destructuring `[& [{:keys [...]}]]` gets a sequence containing one map: `[{:id 1, ...}]`. Then `{:keys [id name email]}` extracts the keys, and all values are non-nil.

### Fix Option 2: Accept Variadic Keyword Arguments

If you want to keep the calling convention (keyword arguments), change the function signature:

```clojure
;; Accept variadic keyword arguments
(defn process [& {:keys [id name email]}]
  (str id ":" name ":" email))

;; Now this works:
(process :id 1 :name "Alice" :email "alice@example.com")
;; => "1:Alice:alice@example.com"  ✓ SUCCESS!
```

**Why this works:** The destructuring `[& {:keys [...]}]` tells Clojure to treat the variadic sequence as keyword arguments directly. It automatically pairs `:id 1 :name "Alice"` etc., into a map internally.

### Fix Option 3: Accept a Regular Map Argument

```clojure
;; Accept a single map (not variadic, not optional)
(defn process [{:keys [id name email]}]
  (str id ":" name ":" email))

;; Call it with a map:
(process {:id 1 :name "Alice" :email "alice@example.com"})
;; => "1:Alice:alice@example.com"  ✓ SUCCESS!
```

**Why this works:** The simplest pattern. No variadic sequence, just destructure the single map argument directly.
```

**Flow:**
1. Test Option 1 (explain why)
2. Test Option 2 (explain why)
3. Test Option 3 (explain why)
4. Later mentions verifying before modifying source

### Analysis

**Improved Skill:**
- ✅ Tests COME FIRST (before talk of updating source)
- ✅ Exact section header: "Test Both Fix Approaches" emphasizes REPL testing
- ✅ Clear separation: "Once you've confirmed... THEN update source"
- ✅ Shows cleanup checklist (remove defs)
- ✅ Focused on 2 most practical options
- ✅ Reads like the intended workflow: test → verify → commit

**Original Skill:**
- ✅ Shows 3 fix options (more comprehensive)
- ✅ Explains WHY each works
- ✓ Tests are shown in REPL
- ✗ Less emphasis on "test BEFORE commit" (more implicit)
- ✗ Doesn't show the cleanup of def statements
- ✓ Educational about different approaches

**Winner for Fix Testing:** **Improved** — Makes it crystal clear that testing happens in REPL FIRST, cleanup happens SECOND, then source updates. The section hierarchy emphasizes this workflow.

---

## 4. OVERALL WORKFLOW STRUCTURE

### Improved Skill
```
Step 1: Reproduce and Inspect Using Inline Def
  → Write function with defs
  → Evaluate in REPL
  → Call function
  → Inspect output
  → Recognize pattern
  → Form hypothesis

Step 2: Test Both Fix Approaches
  → Option A: test in REPL ✓
  → Option B: test in REPL ✓

Step 3: Verify Your Choice and Commit
  → Remove defs
  → Update source
  → Run tests
```

**Strengths:**
- ✅ Linear: "do this, then this, then this"
- ✅ No lookahead required
- ✅ Emphasizes REPL interaction at each step
- ✅ Clear when source file changes happen (last)

### Original Skill
```
Step 1: Reproduce Issue in REPL
Step 2: Inspect What Function Actually Receives
  → Using def to capture
  → Compare expected vs actual
Step 3: Test the Fix in the REPL (Before Modifying Source)
  → Option 1
  → Option 2
  → Option 3
Step 4: Verify Your Understanding Using REPL Inspection
```

**Strengths:**
- ✅ Very thorough
- ✅ 4 distinct phases
- ✓ Also emphasizes REPL-first
- ✗ Step 4 feels like it repeats step 2

---

## Summary Table

| Aspect | Improved | Original | Winner |
|--------|----------|----------|--------|
| **Inline Def Pattern** | Shows add-to-function flow clearly | Shows REPL capture, compares patterns | Improved (+) |
| **Discovery Narrative** | Hypothesis emerges from data | Hypothesis from pattern comparison | Improved (+) |
| **Fix Testing** | Emphasizes "test first" explicitly | Also tests first, but less emphasized | Improved (+) |
| **Code Examples** | Concise, focused | Comprehensive, more options | Tie (different strengths) |
| **Workflow Clarity** | Linear 3-step flow | Detailed 4-step flow | Improved (+) |
| **Length** | 214 lines | 233 lines | Improved (shorter) |
| **Alignment with Borkdude** | Closer (inline def in function) | Good (emphasis on REPL first) | Improved (+) |

**Overall:** Improved skill shows **6 wins, 1 tie** on comparison metrics.

---

## User Feedback Needed

Does the improved skill's approach feel:

1. **More aligned with how you want the inline def pattern taught?** (add to function, evaluate in nrepl, inspect)
2. **Clearer in the discovery narrative?** (hypothesis emerges from what you see, not from pattern understanding)
3. **Better at emphasizing "test in nrepl before touching source"?** (the workflow hierarchy makes this clearer)
4. **More suitable for LLMs to understand and apply consistently?** (shorter, more linear, less optional paths)

**Or** do you prefer the original skill's more comprehensive, exploratory approach?
