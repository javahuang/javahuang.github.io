# react-router

```bash
npm install react-router-dom
```

```jsx
// 基本使用
<Router>
    <nav>
        <ul>
            <li>Home</li>
            <li>About</li>
        </url>
    </nav>
    <Switch>
        <Route path="/about">
            <About />
        </Route>
        <Route path="/users">
            <Users />
        </Route>
    </Switch>
</Router>
```