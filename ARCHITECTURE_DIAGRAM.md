# Architecture Diagram - Run API Integration

## System Flow

```
┌─────────────────────────────────────────────────────────────┐
│                    USER BROWSER                              │
│  http://localhost:3000/problems/102                          │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────────────────────────────────────────────┐   │
│  │         ProblemView Component                         │   │
│  │  - Loads problem statement                           │   │
│  │  - Passes slug to RightCodePanel                    │   │
│  └────────────────────┬─────────────────────────────────┘   │
│                       │                                      │
│                       │ slug="two-sum"                       │
│                       ↓                                      │
│  ┌──────────────────────────────────────────────────────┐   │
│  │      RightCodePanel Component                       │   │
│  │  ┌────────────────────────────────────────────┐    │   │
│  │  │ Language Dropdown                         │    │   │
│  │  │ [Python3 ▼]  [Java ▼]  [JavaScript ▼]    │    │   │
│  │  └────────────────────────────────────────────┘    │   │
│  │                                                     │   │
│  │  ┌────────────────────────────────────────────┐    │   │
│  │  │ Code Editor (Dark Theme)                  │    │   │
│  │  │                                            │    │   │
│  │  │ # cook your dish here                     │    │   │
│  │  │ def solve():                              │    │   │
│  │  │   pass                                     │    │   │
│  │  └────────────────────────────────────────────┘    │   │
│  │                                                     │   │
│  │  [▶ Run] [Submit] [⌚] [☀]                         │   │
│  │                                                     │   │
│  │  ┌────────────────────────────────────────────┐    │   │
│  │  │ Output / Error Panel                       │    │   │
│  │  │ Test Case 1: PASSED ✓                     │    │   │
│  │  │ (or error message in red)                 │    │   │
│  │  └────────────────────────────────────────────┘    │   │
│  │                                                     │   │
│  │  ┌────────────────────────────────────────────┐    │   │
│  │  │ Custom Input                               │    │   │
│  │  │ [textarea]                                 │    │   │
│  │  └────────────────────────────────────────────┘    │   │
│  └──────────────────────┬───────────────────────────────┘   │
│                         │                                   │
│                         │ onClick("▶ Run")                 │
│                         ↓                                   │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  handleRun() Function                              │   │
│  │  1. Extract: slug, code, language                  │   │
│  │  2. Map language: Python3 → python                │   │
│  │  3. Get token from localStorage                    │   │
│  │  4. Set loading = true (disable button)            │   │
│  └────────────────────┬─────────────────────────────────┘   │
│                       │                                      │
│                       │ import { runCode }                  │
│                       ↓                                      │
│  ┌──────────────────────────────────────────────────────┐   │
│  │  practiceAPI.runCode(slug, code, language)          │   │
│  │  ↓                                                   │   │
│  │  fetch('/.../practice/questions/two-sum/run')       │   │
│  │  POST {code, language: "python"}                    │   │
│  │  Headers: Authorization: Bearer TOKEN               │   │
│  └────────────────────┬─────────────────────────────────┘   │
│                       │                                      │
└───────────────────────┼──────────────────────────────────────┘
                        │
                        │ Network Request
                        ↓
┌─────────────────────────────────────────────────────────────┐
│                    BACKEND SERVER                            │
│         http://localhost:5000                               │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  POST /practice/questions/:slug/run                         │
│  ├─ Authenticate user (verify token)                       │
│  ├─ Get question by slug from database                     │
│  ├─ Extract visible test cases (isHidden: false)           │
│  ├─ Execute user code via Piston API                       │
│  │  ├─ Language: python                                   │
│  │  ├─ Code: [user code]                                 │
│  │  ├─ Stdin: [test case input]                          │
│  │  ├─ Timeout: 5 seconds                                │
│  │  └─ Return: output/error                              │
│  │                                                         │
│  └─ Compare output with expected output                    │
│     ├─ Test 1: PASSED ✓                                  │
│     ├─ Test 2: FAILED ✗ (output mismatch)               │
│     └─ Test 3: ERROR (runtime error)                     │
│                                                              │
│  Response (200):                                           │
│  {                                                          │
│    "success": true,                                         │
│    "results": [                                             │
│      {                                                      │
│        "passed": true,                                      │
│        "input": "...",                                      │
│        "expectedOutput": "...",                             │
│        "actualOutput": "...",                               │
│        "error": null                                        │
│      }                                                      │
│    ]                                                        │
│  }                                                          │
│                                                              │
└────────────────────────┬──────────────────────────────────────┘
                         │
                         │ JSON Response
                         ↓
┌─────────────────────────────────────────────────────────────┐
│                    USER BROWSER                              │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ← Response received                                        │
│                                                              │
│  if (data.success && data.results) {                       │
│    ├─ Format results for display                          │
│    ├─ Update runOutput state                              │
│    └─ Set loading = false                                 │
│  }                                                          │
│  else {                                                     │
│    ├─ Set error message                                   │
│    └─ Set loading = false                                 │
│  }                                                          │
│                                                              │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ Test Case Results Display                            │  │
│  │                                                      │  │
│  │ Test Case 1:                                         │  │
│  │ Status: PASSED ✓                                     │  │
│  │ Input: 4\n2 7 11 15\n9                              │  │
│  │ Expected Output: 0 1                                 │  │
│  │ Actual Output: 0 1                                   │  │
│  │                                                      │  │
│  │ ---                                                  │  │
│  │                                                      │  │
│  │ Test Case 2:                                         │  │
│  │ Status: FAILED ✗                                     │  │
│  │ Input: 3\n3 2 4\n6                                  │  │
│  │ Expected Output: 1 2                                 │  │
│  │ Actual Output: 2 1                                   │  │
│  │                                                      │  │
│  └──────────────────────────────────────────────────────┘  │
│                                                              │
│  [▶ Run] button is now enabled again                       │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

## Component Hierarchy

```
App
├── Router
│   └── Routes
│       └── /problems/:id
│           └── ProblemView
│               ├── ProblemNavbar
│               ├── Problem Statement Panel (Left)
│               └── RightCodePanel (Right) ← MODIFIED
│                   ├── Language Selector
│                   ├── Code Editor (textarea)
│                   ├── Buttons [Run] [Submit]
│                   ├── Output Panel
│                   └── Custom Input Panel
```

## Data Flow

```
┌──────────────────┐
│  User Input      │
│ - Writes Code    │
│ - Selects Lang   │
│ - Clicks Run     │
└────────┬─────────┘
         │
         ↓
┌──────────────────────┐
│  RightCodePanel      │
│ - handleRun()        │
│ - Validate inputs    │
│ - Set loading=true   │
└────────┬─────────────┘
         │
         ↓
┌──────────────────────┐
│  practiceAPI         │
│  runCode()           │
│ - Get token          │
│ - Map language       │
│ - Prepare request    │
└────────┬─────────────┘
         │
         ↓
┌──────────────────────┐
│  Fetch to Backend    │
│  POST /practice/...  │
└────────┬─────────────┘
         │
         ↓
┌──────────────────────┐
│  Backend Processing  │
│ - Authenticate       │
│ - Get question       │
│ - Execute via Piston │
│ - Compare outputs    │
└────────┬─────────────┘
         │
         ↓
┌──────────────────────┐
│  Response JSON       │
│ {success, results[]} │
└────────┬─────────────┘
         │
         ↓
┌──────────────────────┐
│  RightCodePanel      │
│ - Format results     │
│ - Update state       │
│ - Set loading=false  │
└────────┬─────────────┘
         │
         ↓
┌──────────────────────┐
│  User Sees Results   │
│ ✓ Passed tests       │
│ ✗ Failed tests       │
│ ⚠ Error messages     │
└──────────────────────┘
```

## State Management

```
RightCodePanel Component State:

┌─ code (string)
│  └─ User's code in editor
│
├─ selectedProgLang (string)
│  └─ Current language: "Python3", "JavaScript", etc.
│
├─ customInput (string)
│  └─ Custom input for testing
│
├─ runOutput (string)
│  └─ Formatted test results
│
├─ loading (boolean) ← NEW
│  └─ true during API call, false otherwise
│
└─ error (string) ← NEW
   └─ Error message if API call fails
```

## Language Mapping

```
UI Language    →  API Language    →  Piston Code
─────────────────────────────────────────────
Python3        →  python          →  "python"
JavaScript     →  javascript      →  "javascript"
Java           →  java            →  "java"
C++            →  cpp             →  "cpp"
Go             →  go              →  "go"
Rust           →  rust            →  "rust"
(and others)
```

## Error Handling Flow

```
handleRun()
├─ No slug provided
│  └─ setError("Problem slug is required")
│
├─ API Call Fails
│  ├─ 401: setError("Please login to run code")
│  ├─ 404: setError("Problem not found")
│  ├─ 400: setError("Validation error...")
│  └─ Other: setError(err.message)
│
├─ Invalid Response
│  └─ setError("Invalid response format")
│
└─ Network Error
   └─ setError(err.message)
```

## Key Files Involved

```
Frontend:
  src/
  ├── components/
  │   └── PracticePage/
  │       └── RecentContext/
  │           └── Problems/
  │               ├── ProblemView.js ← Modified
  │               └── RightCodePanel.js ← Modified
  │
  └── services/
      └── practiceAPI.js ← NEW

Backend:
  src/
  ├── controllers/
  │   └── practiceQuestion.controller.js
  │       └── runCode(req, res)
  │
  ├── routes/
  │   └── practiceQuestion.routes.js
  │       └── POST /:slug/run
  │
  └── services/
      └── codeExecutor.service.js
          └── executeCode()
```

## Success Indicators

✅ Button shows "⏳ Running..." during execution  
✅ Test results display in output panel  
✅ Error messages show in red  
✅ Loading state prevents duplicate submissions  
✅ All test case results visible  

---

**Integration Date:** January 14, 2026  
**Architecture Version:** 1.0  
**Status:** Complete ✓
