# Clean Code Checklist

## Naming

- **Use descriptive and meaningful names**
  - Variables & Properties: nouns or short phrases with adjectives
  - Functions & Methods: verbs or short phrases with adjectives
  - Classes: nouns
- **Be as specific as necessary and as possible**
- **Use yes/no "questions" for booleans** (e.g. `isValid`)
- **Avoid misleading names**
- **Be consistent with your names** (e.g. stick to `get...` instead of switching to `fetch...`)

---

## Comments & Formatting

### Comments
- **Most comments are bad — avoid them!**
- **Some good comments are acceptable:**
  - Legal comments
  - Warnings
  - Helpful explanations (e.g. for regex)
  - TODOs (don't overdo it though)

### Vertical Formatting
- **Keep related concepts close to each other** (vertical density)
- **Add spacing** (e.g. blank lines) between concepts that aren't directly related (vertical distance)
- **Write code top to bottom** — called functions should come below calling functions, if possible

### Horizontal Formatting
- **Avoid long lines** — break them into multiple lines instead
- **Use indentation** to express scope

---

## Functions & Methods

- **Limit the number of parameters** — less is better!
  - Consider using objects, dictionaries, or arrays to group multiple parameters into one
- **Functions should be small and do one thing**
- **Levels of abstraction should be consistent** — one level below what the function name implies
- **Avoid mixing levels of abstraction in functions**
  - *But:* avoid redundant splitting!
- **Stay DRY** (Don't Repeat Yourself)
- **Avoid unexpected side effects**

---

## Control Structures & Errors

- **Prefer positive checks**
- **Avoid deep nesting**
- **Consider using guard statements**
- **Consider using polymorphism and factory functions**
- **Extract control structures into separate functions**
- **Prefer "real" errors over synthetic ones** — use error handling instead of building errors with `if` statements

---

## Objects & Classes

- **Focus on building "real objects" or data containers/structures**
- **Build small classes** — focus on a single responsibility (which does *not* mean "single method"!)
- **Build classes with high cohesion**
- **Follow the Law of Demeter** for "real objects" (avoid `this.customer.lastPurchase.date`)
- **Follow the SOLID principles**, especially when doing OOP
  - SRP and OCP in particular will help a lot with writing clean (readable) code
