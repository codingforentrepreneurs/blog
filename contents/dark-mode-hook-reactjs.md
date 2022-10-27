---
title: Create a Dark Mode hook in React.js
slug: dark-mode-hook-reactjs

publish_timestamp: July 28, 2020
url: https://www.codingforentrepreneurs.com/blog/dark-mode-hook-reactjs/

---

I love dark mode. It's definitely one of my favorite design trends across apps and websites. Below is an example of how the CFE dark mode toggle works.

In this post, we'll create a custom `React.js` hook and context provider to work in any functional component across our projects. 

I'll be using [bootstrap](https://getbootstrap.com) __css__ classes to denote the modes:

- `dark` mode __css__ classes: `bg-dark text-light`
- `light` mode __css__ classes: `bg-light text-dark`

Let's make a quick toggle for our `<body>` tag so we can ensure the `dark` or `light` classes are working globally.

```javascript
const toggleBodyClasses = isDarkMode => {
    if (isDarkMode) {
      // here's a good place to add a dark-mode css classes to our <body> and remove light mode
      document.body.classList.add('bg-dark', 'text-light');
      document.body.classList.remove('bg-light', 'text-dark');
    } else {
      // remove the dark mode classes, add light mode
      document.body.classList.add('bg-light', 'text-dark');
      document.body.classList.remove('bg-dark', 'text-light');
    }

}
```
Naturally, this function isn't enough for us but it is very useful if we want to ensure our entire document is in dark mode. 

We need to create our default context and hook in a file called `DarkMode.js`:

```javascript
import React from 'react'

const defaultContextData = {
  dark: false,
  toggle: () => {}
};

const DarkModeContext = React.createContext(defaultContextData);
export const useDarkMode = () => React.useContext(DarkModeContext);
```

Now, we'll create a custom hook for our dark mode state. 

```javascript

const useEffectDarkMode = () => {
  const [darkModeState, setDarkModeState] = React.useState({
    dark: false,
    hasDarkModeMounted: false
  });
  React.useEffect(() => {
      // remember our current dark mode status in the browser local storage
    const localStorageDark = localStorage.getItem("dark") === "true";
    toggleBodyClasses(localStorageDark)

    setDarkModeState({dark: localStorageDark, hasDarkModeMounted: true});
  }, []);

  return [darkModeState, setDarkModeState];
};
```

Now, we'll make a context provider for our app. This is the only place we'll need the `useEffectDarkMode` hook.

```javascript
export const DarkModeProvider = ({ children }) => {
  const [darkModeState, setDarkModeState] = useEffectDarkMode();

  if (!darkModeState.hasDarkModeMounted) {
    return <div />;
  }

  const toLightMode = () => {
    if (darkModeState.dark) {
      toggle()
    }
  }
  const toDarkMode = () => {
    if (!darkModeState.dark) {
      toggle()
    }
  }
  const toggle = () => {
    const dark = !darkModeState.dark;
    // remember our current dark mode status in the browser local storage
    localStorage.setItem("dark", JSON.stringify(dark));
    
    toggleBodyClasses(dark)

    setDarkModeState({ ...darkModeState, dark, bgColor:lsBgColor, textColor:lsTextColor });
  };
  return (
      <DarkModeContext.Provider
        value={{
          dark: darkModeState.dark,
          toDark: toDark,
          toLight: toLight,
          toggle: toggle
        }}
      >
        {children}
      </DarkModeContext.Provider>
  );
};
```

Now, let's add our provider to `index.js`. I put the `DarkModeProvider` on `index.js` to ensure that my entire `App` has access to the `DarkModeContext` via the `useDarkMode` hook


```javascript
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';
import { DarkModeProvider } from './DarkMode';
ReactDOM.render(
<DarkModeProvider>
    <App />
</DarkModeProvider>, 
document.getElementById('root'));
```

Now let's look at an example of how we can use it, let's say on `Navbar.js`

```javascript
import React from 'react'

import {useDarkMode} from './DarkMode'

export const Navbar = (props) => {
    const darkModeState = useDarkMode()
    const navbarBgClass = darkModeState.dark ? 'bg-dark' : 'bg-light'
    return <nav className={`navbar navbar-light ${navbarBgClass}`}>
    <div className="collapse navbar-collapse">
        <ul className="navbar-nav mr-auto">
            <li className="nav-item">
                <span className="nav-link" onClick={e=>darkModeState.toggle()}>Toggle</span>
            </li>
            <li className="nav-item">
                <span className="nav-link" onClick={e=>darkModeState.toDark()}>To Dark</span>
            </li>
            <li className="nav-item">
                <span className="nav-link" onClick={e=>darkModeState.toLight()}>To Light</span>
            </li>
        </ul>
    </div>
</nav>
}
```

And maybe a re-usable button
```javascript
import React from 'react'
import { useDarkMode } from './DarkMode'

export const DarkModeToggleButton = (props) => {
    const darkModeState = useDarkMode()
    return <button className={props.className} onClick={() => darkModeState.toggle()}>
        {darkModeState.dark ? "Dark mode on" : "Dark mode off"}      
    </button>
}
```

Pretty neat huh?