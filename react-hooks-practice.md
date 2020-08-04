接触react-hook已经很久了，用了之后，发现真的很强大，记录一下最近使用的想法

### useState
首先接触到的就是`useState`
state只是一个状态，但是之前class就需要写的很大，相比之下，写成hook就简化了很多
```jsx
class Age extends React.Component {
  constructor(props) {
    super(props);

    this.state = {
      age: 20
    };
  }

  handleOnChange = e => {
    const { name, value } = e.target;
    this.setState({ [name]: value });
  };

  render() {
    return (
      <>
        <div>age: {this.state.age}</div>
        <input name="age" onChange={handleOnChange} />
      </>
    );
  }
}
```
用了hook之后，就变得很简洁了
```jsx
const Age = () => {
  const [age, setAge] = React.useState(20);

  const handleOnChange = e => {
    const { value } = e.target;
    setAge(value);
  };

  return (
    <>
      <div>age: {age}</div>
      <input name="age" onChange={handleOnChange} />
    </>
  );
};
```
### useEffect
对于useEffect我最初以为是用来替代生命周期，比如`componentDidMount`，但是事实证明他真的很强大，`[name]`括号里面的是依赖，依赖一变，就会重新执行，多个依赖就更棒了，如果是用之前的写法，就要在 `componentWillReceiveProps`里面写很多if来比较`nextProps`和`this.props`
而且这个最强大就是可以抽出来，当一个反复利用reuseable的hook

传入不同的`localStorageKey`，就可以存不同的数据到`localStorage`， 这样就方便重复实用了，而且每次`value`变了，就会自动保存到`localStorage`
```jsx
const useLocalStorage = localStorageKey => {
  const [value, setValue] = React.useState(
    localStorage && localStorage.getItem(localStorageKey)
  );

  React.useEffect(() => {
    localStorage && localStorage.setItem(localStorageKey, value);
  }, [value, localStorageKey]);

  return [value, setValue];
};
```
### useRef
 有些时候，会发现，其实数据并没有变化，但是还是触发了setState或者是别的callback，这样会导致，调用的函数太多，从而影响了性能，所以比较两个对象是不是相等，只有当不等，才更新，这样可以减小一点re-render。 
这时候，就是useRef登场的时候了，搭配实用lodash.isEqual, 可以比较大的object
```jsx
const usePrevious = value => {
  const ref = useRef();

  useEffect(() => {
    ref.current = value;
  }, [value]);

  return ref.current;
};

#使用
const myPreviousState = usePrevious(conditions);
useEffect(() => {
 if (myPreviousState && !_.isEqual(myPreviousState, conditions)) {
        onChange(conditions);
 }
}, [myPreviousState, conditions, onChange ])
```
### useContext
context呢，我之前一直觉得这个东西很高大上，很复杂，一直不敢用，但是后来发现，他就是对应上下文，比如传递props，一般都是使用parent传给child，但是如果，A -> B -> C -> D，但其实B和C都没用到，只有D用到了，那么D要来修改A，就需要再往上传递，导致B和C一直在反反复复被重新渲染，这时候，当当当当，context就可以发挥很大的作用，在A里面包一下Provider，然后到D直接用`createContext`和`useContext`拿出来就可以用了，这样代码也不会忘了忘记中间的B，C的pass props，看起来也清楚，何乐而不为呢
```jsx
const NameContext = createContext();

const A = () => (
   <NameContext.Provider value={{ username, setUserName }}>
     <B />
   </NameContext.Provider>
);

const Names = () => {
  const { username, setUserName } = useContext(NameContext);

  const { names, onChange } = useNames({ username, setUserName });

  return (
    <div>
      <label>firstName</label>
      <input
        name="firstName"
        value={names && names.firstName}
        onChange={onChange}
      />
      <label>lastName</label>
      <input
        name="lastName"
        value={names && names.lastName}
        onChange={onChange}
      />
    </div>
  );
}; 
```
### useReducer
最后是用到redux, 之前对于redux的理解一直如果当一个对象，要被很多不同的地方共享和修改，那么放到redux是最合适不过。但是知道我遇到了，一个嵌套很深的一个object，要更新里面的某一个字段真的要写好多好多层，然后我翻到了hook对于useReducer的介绍

> Another option is useReducer, which is more suited for managing state objects that contain multiple sub-values.

> useReducer is usually preferable to useState when you have complex state logic that involves multiple sub-values or when the next state depends on the previous one. useReducer also lets you optimize performance for components that trigger deep updates because [you can pass dispatch down instead of callbacks](https://reactjs.org/docs/hooks-faq.html#how-to-avoid-passing-callbacks-down)

现在的reducer是相对于useState而言的，他更加适合复杂的，有很多sub-values的对象的更新
```jsx
import React, { useEffect, useContext, createContext, useReducer } from "react";
const NameContext = createContext();

const init = initial => {
  return { name: initial };
};

const reducer = (state, action) => {
  switch (action.type) {
    case "firstName":
      return {
        ...state,
        name: {
          firstName: action.payload
        }
      };
    case "lastName":
      return {
        ...state,
        name: {
          lastName: action.payload
        }
      };
    default:
      return state;
  }
};

const useNames = ({ username, setUserName }) => {
  // const [names, setNames] = React.useState(username);
  const [state, dispatch] = useReducer(reducer, username, init);
  const names = state.name;

  const onChange = e => {
    const { name, value } = e.target;
    dispatch({ type: name, payload: value });
  };

  useEffect(() => {
    setUserName(names);
  }, [names, setUserName]);

  return {
    names,
    onChange
  };
};

const Names = () => {
  const { username, setUserName } = useContext(NameContext);
  const { names, onChange } = useNames({ username, setUserName });
  return (
    <div>
      <label>firstName</label>
      <input
        name="firstName"
        value={names && names.firstName}
        onChange={onChange}
      />
      <label>lastName</label>
      <input
        name="lastName"
        value={names && names.lastName}
        onChange={onChange}
      />
    </div>
  );
};

const App = () => {
  const [username, setUserName] = React.useState({
    firstName: "Rachel",
    lastName: "Green"
  });

  return (
    <NameContext.Provider value={{ username, setUserName }}>
      <Names />
    </NameContext.Provider>
  );
};

export default App;
```
完成了一个简简单单的name 表单组
![image.png](https://upload-images.jianshu.io/upload_images/1163658-e3160676f81c1a7a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
