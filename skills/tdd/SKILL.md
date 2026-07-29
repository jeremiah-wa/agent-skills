---
name: tdd
description: "Build test-first: write the failing test, watch it fail for the right reason, make it pass, then clean up. Use when implementing a behavioural change or fixing a bug."
---

Write the test first, and **watch it fail** before you make it pass.

The order is not a ritual. A test you wrote after the code passes because the code exists, and you never learn whether it would have caught anything. A test you watched fail has proven it can.

## The loop

**Red.** Write one test for one behaviour that does not exist yet. Run it. Read the failure.

The failure must be the one you expected. A test failing on an import error, a typo, or a missing fixture has told you nothing about the behaviour. Fix the test until it fails for the right reason, then continue.

**Green.** Write the least code that makes it pass. Run the test again.

Not the best code. The least. Getting to green fast is what tells you the test is wired to the thing you think it is wired to.

**Refactor.** Now clean it up, with the test holding the behaviour still. Run it again after.

Then take the next behaviour. One at a time.

## Test at real seams

A **seam** is a boundary where the code can be observed without reaching inside it: a function's return, a rendered output, a written record, an emitted event, a response body.

Test there. Behaviour at a seam survives refactoring, because refactoring is precisely the act of changing everything except behaviour at a seam.

The failure mode is a test bound to the shape of the implementation rather than to what it does: asserting a private method was called, a field was set, a mock received a message. Those tests break on every rewrite and catch nothing, so they train you to stop trusting the suite.

When you cannot see the behaviour from outside, that is a design finding, not a testing problem. Say so, and consider whether the seam is missing.

## When there is no seam

Some changes genuinely have nowhere to attach a test: configuration, a migration, dependency plumbing, a rename.

**Say so plainly** and move on. Name the change and why no test applies, in the commit body or the PR body. That sentence is worth more to a reviewer than a test asserting that a config file contains the value it was just given.

Do not invent a test to fill the slot. A test that cannot fail is a false green, and it costs a reader more than the empty slot would.

## What a test is worth

The bar is a **failure scenario**: specific inputs or state leading to a specific wrong outcome, which this test would catch.

If you cannot state one, the test is decoration. Delete it and write the one you can state.

Prefer the inputs nobody considered: empty, zero, negative, missing, enormous, out of order, arriving twice. The happy path is already exercised by everything else in the system.

## Bugs

A bug fix starts with a test that **reproduces** it, run before the fix and observed failing.

That is the whole discipline. If you cannot reproduce it, you do not yet know what you are fixing, and the fix is a guess that will be re-fixed later. Getting to the reproduction is the work; the fix is usually small.
