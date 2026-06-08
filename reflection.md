# 💭 Reflection: Game Glitch Investigator

Answer each question in 3 to 5 sentences. Be specific and honest about what actually happened while you worked. This is about your process, not trying to sound perfect.

## 1. What was broken when you started?

- What did the game look like the first time you ran it?
  This is a number guessing game with three difficulty levels: Easy, Normal, and Hard. Each level gives you a limited number of attempts to guess a secret number within a certain range. If you guess correctly within the allowed attempts, you win. When I first ran it, the game appeared to work on the surface, but several things felt immediately wrong. The difficulty levels did not make sense, the hints pointed me in the wrong direction, and the New Game button did not fully reset the game.

- List at least two concrete bugs you noticed at the start:
  - When my guess was 1, the game told me to "Go LOWER!", the opposite of what it should say.
  - Normal ranged from 1–100, but Hard only ranged from 1–50, making Hard an easy mode.
  - Clicking "New Game" after losing still left the game in a "lost" state, so I could not guess again without refreshing the page.
  - No matter which difficulty I selected, the hint banner always said "Guess a number between 1 and 100," ignoring the actual range.

**Bug Reproduction Log**

Document at least 3 bugs you found. Add rows as needed.

| Input                                     | Expected Behavior                                               | Actual Behavior                                                                                                                 | Console Output / Error                                       |
| ----------------------------------------- | --------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------ |
| Secret = 42, Guess = 10 (attempt 1, odd)  | Hint says "Go Higher" since 10 < 42                             | Hint says "📉 Go LOWER!", hint direction is reversed                                                                            | No error; wrong message displayed                            |
| Secret = 42, Guess = 10 (attempt 2, even) | Hint says "Go Higher" since 10 < 42                             | Secret is cast to the string `"42"`, so `"10" > "42"` lexicographically, hint says "Go HIGHER!" for the wrong reason            | No error; string comparison produces misleading result       |
| Lose a game, then click "New Game 🔁"     | Game resets to "playing" state; player can submit guesses again | `st.session_state.status` is never reset; game stays in "lost" state and immediately blocks with "Game over. Start a new game." | No error; `st.stop()` is hit before guesses can be submitted |
| Select "Hard" difficulty                  | Harder range than Normal (e.g., 1–200)                          | Range is 1–50, which is narrower and easier than Normal's 1–100                                                                 | No error; sidebar shows "Range: 1 to 50"                     |

---

## 2. How did you use AI as a teammate?

- Which AI tools did you use on this project (for example: ChatGPT, Gemini, Copilot)?
  I used Claude Code as my AI assistant throughout this project.
- Give one example of an AI suggestion that was correct (including what the AI suggested and how you verified the result).
  I asked Claude to identify the hint bug. It correctly spotted that check_guess in app.py had the return strings swapped. "Too High" was paired with "Go HIGHER!" and "Too Low" with "Go LOWER!". I verified this by tracing through the function manually: when guess > secret, the player needs to go lower, so the fix was to return "Go LOWER!" there instead. Running the game after the fix confirmed hints pointed in the right direction.
- Give one example of an AI suggestion that was incorrect or misleading (including what the AI suggested and how you verified the result).
  There was actually no incorrect or misleading while I used AI for this project.

---

## 3. Debugging and testing your fixes

- How did you decide whether a bug was really fixed?
  I ran streamlit run app.py and played the game manually to confirm the fix worked in the live UI.
- Describe at least one test you ran (manual or using pytest)  
  and what it showed you about your code.
  I ran pytest tests after refactoring check_guess into logic_utils.py. The tests revealed that the original starter tests were comparing against a plain string like "Win", but check_guess actually returns a tuple ("Win", "🎉 Correct!"). This told me the tests themselves were broken, not just the game logic. I updated them to unpack the tuple with outcome, message = check_guess(...) and added two extra assertions to verify the hint text said "LOWER" or "HIGHER" in the right cases.
- Did AI help you design or understand any tests? How?
  Yes. I asked Claude to identify why the starter tests were failing. It spotted the tuple vs string mismatch immediately and suggested unpacking the return value. I verified this was correct by reading the check_guess function in logic_utils.py and confirming it always returns a two-element tuple.

---

## 4. What did you learn about Streamlit and state?

- How would you explain Streamlit "reruns" and session state to a friend who has never used Streamlit?
  Every time you interact with the app, Streamlit reruns the entire script from top to bottom, so regular variables reset on every click. st.session_state is a persistent dictionary that survives those reruns. I learned this the hard way with the New Game bug. The game stayed locked after a loss because status was never reset before Streamlit hit st.stop() on the next rerun.

---

## 5. Looking ahead: your developer habits

- What is one habit or strategy from this project that you want to reuse in future labs or projects?
  - This could be a testing habit, a prompting strategy, or a way you used Git.
    Habit I want to reuse: I'll always add # FIXME comments before touching any code. Marking the exact line where the bug lives gave me a precise anchor when prompting the AI, instead of describing the problem vaguely, I could point directly to the broken line.
- What is one thing you would do differently next time you work with AI on a coding task?
  I think I can ask AI to point out bugs in the codebase first before in test them in the UI. So I can verify if the AI detects the bugs correctly.
- In one or two sentences, describe how this project changed the way you think about AI generated code.
  For this project, it's pretty good tho and made no mistakes.
