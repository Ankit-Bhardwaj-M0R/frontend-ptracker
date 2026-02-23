# React Learning Roadmap for This Codebase

Let me give you a step-by-step approach to understanding this Performance Management System. I'll order it from simple to complex concepts.

---

## **PHASE 1: Foundation (Start Here)**

### 1. **Understand the Project Structure First**
```
frontend2/
├── index.html              # The single HTML page
├── src/
│   ├── main.jsx            # Entry point - mounts React app
│   ├── App.jsx             # Main component with all routes
│   ├── index.css           # Global styles + Tailwind
│   ├── components/         # Reusable UI pieces
│   ├── pages/              # Full pages (each URL)
│   ├── context/            # Global state management
│   └── services/           # API calls to backend
```

### 2. **Start with the Entry Point**
Open `src/main.jsx` - it's just 5 lines:
```jsx
ReactDOM.createRoot(...).render(
  <React.StrictMode>
    <App />                  // Your whole app starts here
  </React.StrictMode>
)
```
This tells React: "Render the App component into the HTML element with id 'root'"

---

## **PHASE 2: Routing - How Pages Connect**

### 3. **Open `App.jsx` - This is Your Roadmap**
Think of `App.jsx` as a **routing table** that maps URLs to components:

```jsx
<Routes>
  <Route path="/login" element={<LoginPage />} />
  <Route path="/dashboard" element={<ProtectedRoute><SmartDashboard /></ProtectedRoute>} />
  <Route path="/goals" element={<ProtectedRoute><EmployeeGoalsPage /></ProtectedRoute>} />
  <Route path="/team-goals" element={<ProtectedRoute roles={['MANAGER']}><TeamGoalsPage /></ProtectedRoute>} />
  // ... more routes
</Routes>
```

**Key Insight**: When you type `/login` in the browser, React renders `LoginPage`. When you type `/goals`, it renders `EmployeeGoalsPage` (but only if logged in).

---

## **PHASE 3: Start with a Simple Page**

### 4. **Examine `LoginPage.jsx` First**
This is the simplest page and teaches core React concepts:

```jsx
export default function LoginPage() {
  // 1. STATE - data that changes
  const [email, setEmail] = useState('')
  const [password, setPassword] = useState('')
  const [loading, setLoading] = useState(false)
  
  // 2. EVENT HANDLER - what happens when form submitted
  const handleSubmit = async (e) => {
    e.preventDefault()        // Stop page refresh
    setLoading(true)          // Show loading spinner
    try {
      await login(email, password)  // Call API
      navigate('/dashboard')        // Go to dashboard
    } catch (err) {
      setError('Login failed')
    }
  }
  
  // 3. JSX - what the user sees
  return (
    <form onSubmit={handleSubmit}>
      <input value={email} onChange={(e) => setEmail(e.target.value)} />
      <button disabled={loading}>Login</button>
    </form>
  )
}
```

**React Core Concepts Here**:
- **useState** → stores data that changes
- **Event handlers** → respond to user actions
- **JSX** → HTML-like syntax for UI

---

## **PHASE 4: Layout & Components**

### 5. **Understand the Layout Structure**
Most pages are wrapped in a `Layout` component:

```jsx
// Inside any page, e.g., GoalsPage.jsx
return (
  <Layout title="Goals Management">
    {/* page content here */}
  </Layout>
)
```

Now open `components/layout/Layout.jsx`:
```jsx
export default function Layout({ children, title }) {
  return (
    <div>
      <Sidebar />            {/* Left navigation - always visible */}
      <div>
        <Header title={title} />  {/* Top bar with user info */}
        <main>{children}</main>    {/* Your page content goes here */}
      </div>
    </div>
  )
}
```

**Key Insight**: `children` is a special prop - whatever you put between `<Layout>` tags gets passed here.

---

## **PHASE 5: Context - Global Data**

### 6. **Understand `AuthContext.jsx` - Who is logged in?**
This is like a **global variable** that any component can access:

```jsx
// Step 1: Create context (empty container)
const AuthContext = createContext(null)

// Step 2: Provider that wraps the whole app (in App.jsx)
export function AuthProvider({ children }) {
  const [user, setUser] = useState(null)
  
  const login = async (email, password) => {
    const userData = await api.login(email, password)
    setUser(userData)                    // Update global state
    localStorage.setItem('user', JSON.stringify(userData))  // Save to browser
  }
  
  return (
    <AuthContext.Provider value={{ user, login }}>
      {children}
    </AuthContext.Provider>
  )
}

// Step 3: Custom hook for easy access
export function useAuth() {
  return useContext(AuthContext)
}
```

**Now any component can do:**
```jsx
function SomeComponent() {
  const { user, login } = useAuth()  // Gets the global data
  return <div>Welcome {user?.name}</div>
}
```

---

## **PHASE 6: Services - Talking to Backend**

### 7. **Look at `services/api.js` - The Base Setup**
```jsx
const api = axios.create({
  baseURL: '/api/v1',        // All requests start with this
})

// Automatically add token to every request
api.interceptors.request.use(config => {
  const token = localStorage.getItem('token')
  if (token) config.headers.Authorization = `Bearer ${token}`
  return config
})
```

### 8. **Then a Specific Service (e.g., `goalService.js`)**
```jsx
const goalService = {
  getGoals: async (page = 0) => {
    const response = await api.get('/goals', { params: { page } })
    return response.data      // Already unwrapped by api.js
  },
  
  createGoal: async (goalData) => {
    return await api.post('/goals', goalData)
  }
}
```

**How they're used in pages:**
```jsx
// Inside GoalsPage.jsx
import goalService from '../services/goalService'

const loadGoals = async () => {
  const data = await goalService.getGoals()
  setGoals(data)  // Update state with fetched goals
}
```

---

## **PHASE 7: Build Up to a Complex Page**

### 9. **Now Study `pages/employee/EmployeeGoalsPage.jsx`**
This is the most complex page and combines everything:

**What it does**:
- Shows list of goals
- Lets employees create goals
- Lets managers approve/reject
- Has multiple modal forms

**Structure to focus on**:

```jsx
export default function GoalsPage() {
  // 1. Get global data
  const { user, isManager } = useAuth()
  
  // 2. Local state
  const [goals, setGoals] = useState([])
  const [loading, setLoading] = useState(true)
  const [showCreateModal, setShowCreateModal] = useState(false)
  
  // 3. Load data on mount
  useEffect(() => { loadGoals() }, [])
  
  // 4. API calls
  const loadGoals = async () => {
    const data = await goalService.getGoals()
    setGoals(data)
  }
  
  // 5. Render JSX
  return (
    <Layout title="My Goals">
      <button onClick={() => setShowCreateModal(true)}>New Goal</button>
      
      {loading ? <LoadingSpinner /> : (
        goals.map(goal => <GoalCard key={goal.goalId} goal={goal} />)
      )}
      
      {showCreateModal && (
        <Modal onClose={() => setShowCreateModal(false)}>
          <form onSubmit={handleCreateGoal}>...</form>
        </Modal>
      )}
    </Layout>
  )
}
```

---

## **PHASE 8: Follow a Complete Flow**

### 10. **Trace What Happens When You Click "Create Goal"**

1. **User clicks button** → `setShowCreateModal(true)`
2. **Modal appears** with form
3. **User fills form, clicks Submit** → `handleCreateGoal` runs
4. **Inside handleCreateGoal**:
   ```jsx
   const handleCreateGoal = async (e) => {
     e.preventDefault()
     setSubmitting(true)  // Show loading
     try {
       await goalService.createGoal(goalForm)  // API call
       toast.success('Goal created!')           // Show success
       setShowCreateModal(false)                 // Close modal
       loadGoals()                                // Refresh list
     } catch (err) {
       toast.error('Failed')  // Show error
     }
   }
   ```
5. **goalService.createGoal** sends POST to `/api/v1/goals`
6. **Backend saves**, returns success
7. **Page refreshes list**, shows new goal

---

## **QUICK REFERENCE: What Each File Type Does**

| File | Purpose | Example |
|------|---------|---------|
| `pages/*.jsx` | Full screens (what user sees at each URL) | `LoginPage`, `GoalsPage` |
| `components/*.jsx` | Reusable UI pieces | `Modal`, `StatusBadge` |
| `context/*.jsx` | Global state shared everywhere | `AuthContext` (user data) |
| `services/*.js` | API calls to backend | `goalService.js` |
| `App.jsx` | Routing - maps URLs to pages | Which page for `/goals`? |

---

## **SUGGESTED READING ORDER**

1. `src/main.jsx` → `src/App.jsx` (routing overview)
2. `src/pages/LoginPage.jsx` (simplest page)
3. `src/context/AuthContext.jsx` (global user state)
4. `src/services/api.js` + `src/services/authService.js` (API calls)
5. `src/components/layout/Layout.jsx` (how pages are wrapped)
6. `src/pages/employee/EmployeeGoalsPage.jsx` (complex example)
7. `src/components/common/Modal.jsx` (reusable component)
8. `src/context/NotificationContext.jsx` (advanced real-time)

---

## **KEY REACT CONCEPTS TO MASTER**

1. **useState** → `const [count, setCount] = useState(0)`
2. **useEffect** → `useEffect(() => { loadData() }, [])` (runs once on mount)
3. **useContext** → `const { user } = useAuth()` (get global data)
4. **Props** → `<Modal title="Hello">` passes data to child
5. **Conditional rendering** → `{loading && <Spinner />}`
6. **Mapping lists** → `goals.map(goal => <GoalCard key={goal.id} />)`

Start with #1-3, then build up! 🚀