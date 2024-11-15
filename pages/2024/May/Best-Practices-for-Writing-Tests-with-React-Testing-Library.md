## 🔗 [Best Practices for Writing Tests with React Testing Library](https://claritydev.net/blog/improving-react-testing-library-tests)

### 🗓️ 번역 날짜: 2024.05.31

### 🧚 번역한 크루: 마스터위(명재위)

---

Updated On March 12, 2024
Keyword : **React**, **React-Testing-Library**, **Testing**

## React Testing Library로 테스트를 작성하는 모범 사례

[React 테스트 라이브러리](https://testing-library.com/docs/react-testing-library/intro/)는 React 컴포넌트 테스트를 위한 사실상의 표준이 되었습니다. 사용자 관점에서의 테스트에 집중하고, 테스트에서는 구현 세부 사항을 감추는 것이 성공의 주요 이유 중 일부입니다.

올바르게 작성된 테스트는 regressions와 버그가 있는 코드를 방지하는 데 도움이 될 뿐만 아니라, React Testing Library의 경우 [컴포넌트의 접근성](https://claritydev.net/blog/creating-accessible-form-components-with-react)과 전반적인 사용자 경험을 향상시킵니다. React 컴포넌트로 작업할 때는 적절한 테스트 기법을 사용하여 React 테스트 라이브러리에서 흔히 발생하는 실수를 피하는 것이 중요합니다.

이 글에서는 특정 쿼리를 사용하여 테스트 커버리지를 개선하는 방법, `*ByRole` 쿼리의 중요성, 사용자 상호작용 시뮬레이션을 위해 `fireEvent`보다 `userEvent` 메서드를 사용하는 방법, 비동기 테스트를 위해 `findBy*` 쿼리와 `waitForElementToBeRemoved`를 사용하는 방법과 같은 주제와 함께 React 테스트 라이브러리를 사용할 때 가장 흔히 저지르는 실수 몇 가지를 다룰 것입니다. 이 글이 끝나면 더 나은 React 테스트 라이브러리 테스트를 작성하고, 일반적인 실수를 피하고, React 애플리케이션의 전반적인 품질을 개선할 수 있는 지식을 갖추게 될 것입니다.

### Default to `*ByRole` queries

React 테스트 라이브러리의 강력한 장점 중 하나는 올바른 쿼리를 사용하면 컴포넌트가 예상대로 작동할 뿐만 아니라 [접근성](https://claritydev.net/blog/creating-accessible-form-components-with-react)도 보장할 수 있다는 것입니다. 그렇다면 어떤 쿼리가 가장 좋은지 어떻게 알아낼 수 있을까요? 규칙은 매우 간단합니다. 기본적으로 `*ByRole` 쿼리를 사용하세요. 이 쿼리는 많은 요소, 심지어 [복잡한 select components](https://claritydev.net/blog/testing-select-components-react-testing-library)에서도 작동합니다.

대부분의 규칙과 마찬가지로 모든 HTML 요소에 기본 역할이 있는 것은 아니므로 예외가 있습니다. HTML 요소의 기본 역할 목록은 [w3.org](https://www.w3.org/TR/html-aria/#docconformance)에서 확인할 수 있습니다.

### Testing form components

[이전의 튜토리얼 글](https://claritydev.net/blog/typescript-typing-form-events-in-react)에서 변형한 다음 컴포넌트를 보겠습니다.

```jsx
export const Form = ({ saveData }) => {
  const [state, setState] = useState({
    name: "",
    email: "",
    password: "",
    confirmPassword: "",
    conditionsAccepted: false,
  });

  const onFieldChange = (event) => {
    let value = event.target.value;
    if (event.target.type === "checkbox") {
      value = event.target.checked;
    }

    setState({ ...state, [event.target.id]: value });
  };

  const onSubmit = (event) => {
    event.preventDefault();
    saveData(state);
  };

  return (
    <form className='form' onSubmit={onSubmit}>
      <div className='field'>
        <label>Name</label>
        <input
          id='name'
          onChange={onFieldChange}
          placeholder='Enter your name'
        />
      </div>
      <div className='field'>
        <label>Email</label>
        <input
          type='email'
          id='email'
          onChange={onFieldChange}
          placeholder='Enter your email address'
        />
      </div>
      <div className='field'>
        <label>Password</label>
        <input
          type='password'
          id='password'
          onChange={onFieldChange}
          placeholder='Password should be at least 8 characters'
        />
      </div>
      <div className='field'>
        <label>Confirm password</label>
        <input
          type='password'
          id='confirmPassword'
          onChange={onFieldChange}
          placeholder='Enter the password once more'
        />
      </div>
      <div className='field checkbox'>
        <input type='checkbox' id='conditions' onChange={onFieldChange} />
        <label>I agree to the terms and conditions</label>
      </div>
      <button type='submit'>Sign up</button>
    </form>
  );
};
```

form elements를 통해 데이터 입력을 시뮬레이션하고 양식을 제출한 다음 `saveData` prop가 입력한 데이터를 수신했는지 확인하여 이를 테스트할 수 있습니다. 이를 3단계로 나눌 수 있습니다:

1. 테스트할 필드에 텍스트를 입력하거나 확인란을 클릭합니다.
2. `Sign up` 버튼을 클릭합니다.
3. 입력한 데이터로 `saveData`가 호출되었는지 확인합니다.

이 workflow는 사용자가 form과 상호 작용하는 방식을 매우 유사하게 반영합니다(저장된 데이터를 동일한 방식으로 검사하지는 않을 수 있음).

### Querying by placeholder text

첫 번째 입력 필드에 이름을 입력하는 것으로 시작하겠습니다. name placeholder임을 적었으므로, 해당 Inputd을 이를 사용하여 querying 하면 어떨까요?

```jsx
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import "@testing-library/jest-dom/extend-expect";

const defaultData = {
  conditionsAccepted: false,
  confirmPassword: "",
  email: "",
  name: "",
  password: "",
};

describe("Form", () => {
  it("should submit correct form data", async () => {
    const user = userEvent.setup();
    const mockSave = jest.fn();
    render(<Form saveData={mockSave} />);

    await user.type(screen.getByPlaceholderText("Enter your name"), "Test");
    await user.click(screen.getByText("Sign up"));

    expect(mockSave).toHaveBeenLastCalledWith({ ...defaultData, name: "Test" });
  });
});
```

이 방법도 효과가 있지만 그보다 더 나은 방법이 있습니다.
첫째, 이 접근 방식은 placeholder text를 label로 사용하는 관행을 조장할 수 있으며, 이는 placeholder의 원래 용도가 아니면서 [W3C WAI](https://www.w3.org/WAI/tutorials/forms/instructions/#placeholder-text)에서 권장하지 않습니다. 둘째, 접근성 문제를 염두에 두고 테스트하지 않습니다.

### Querying specific components by role

대신 쿼리를 `getByRole`로 대체해 보겠습니다. 문서에 나와 있듯이 `text box`의 역할에 따라 `text` 유형의 입력을 일치시킬 수 있습니다. 하지만 양식에 여러 개의 text box가 있으므로 이보다 더 구체적이어야 합니다. 다행히도 쿼리는 옵션 객체인 두 번째 매개 변수를 허용하므로 `name` 속성을 사용하여 일치하는 항목의 범위를 좁힐 수 있습니다.

[RTL 공식문서](https://testing-library.com/docs/queries/byrole/)를 보면 여기서 `name`이 입력의 이름 속성이 아니라 [accessible `name`](https://www.tpgi.com/what-is-an-accessible-name/)을 가리킨다는 것을 알 수 있습니다. 따라서 입력의 경우 accessible name은 레이블의 텍스트 콘텐츠인 경우가 많습니다. 우리 양식에서는 `name input`에 Name label이 있으므로 이를 사용하겠습니다.

```jsx
user.type(screen.getByRole("textbox", { name: "Name" }), "Test");
```

근데 테스트를 돌리면 다음과 같은 에러가 발생합니다:

```text
TestingLibraryElementError: Unable to find an accessible element with the role "textbox" and name "Name"
```

아래의 `help text`는 입력에 엑세스할 수 있는 이름이 없다는 것을 보여줍니다.

```text
Here are the accessible roles:
  textbox:
  Name "":
  <input
    id="name"
    placeholder="Enter your name"
  />
```

input에 대한 label이 있는데 왜 작동하지 않을까요? [label을 input과 연결](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/label)해야 하는 것으로 밝혀졌습니다. 이를 위해서는 label에 연결된 input의 `ID`와 일치하는 `for` 속성이 있어야 합니다. 또는 input element를 label 안에 래핑할 수도 있습니다. input에 이미 id가 있는 것 같으므로 for(React를 사용하는 경우 `htmlFor`) 속성을 추가하기만 하면 됩니다:

```jsx
<label htmlFor="name">Name</label>
<input
  id="name"
  onChange={onFieldChange}
  placeholder="Enter your name"
/>
```

이제 input이 label과 올바르게 연결되고 테스트가 통과됩니다. 또한 접근성도 크게 향상됩니다.
첫째, label을 클릭하거나 탭하면 focus가 연결된 input으로 맞춰집니다. 둘째, 가장 중요한 것은 input에 초점이 맞춰지면 screen readers가 label을 읽어 사용자에게 input에 대한 추가 정보를 제공한다는 점입니다. 이는 `getByRole`로 전환하면 테스트 커버리지가 향상될 뿐만 아니라 form component에 대한 접근성이 어떻게 개선되는지 보여줍니다.

### Improving the button test

테스트를 다시 살펴보니 submit button에 `getByText` 쿼리가 사용된 것을 확인할 수 있습니다. 제 생각에는 `*ByText`가 가장 깨지기 쉽기 때문에 최후의 수단으로(또는 `*ByTestId`보다 뒤에서 두번째로) 사용해야 합니다.

이 테스트에서 `screen.getByText("Sign up")`는 해당 element를 **Sigh up** text content가 있는 텍스트 노드와 match시킵니다. 나중에 같은 페이지에 "Sign up"이라는 텍스트가 있는 단락을 추가하기로 결정하면 해당 element도 match하게 되고, 여러 개의 matching element로 인해 테스트가 중단됩니다. 문자열 대신 일반 정규식을 텍스트 일치에 사용하면 상황이 더 악화됩니다: `screen.getByText(/Sign up/i)`. 이 경우 대소문자에 관계없이 'sign up'이라는 문자열이 더 큰 문장의 일부이더라도 모두 일치합니다.

이 특정 문자열만 일치하도록 정규식을 수정할 수도 있지만, 대신 더 정확한 쿼리를 사용하면서 동시에 `*ByRole` 쿼리를 사용하여 form에 액세스할 수 있는지 확인할 수 있습니다. 이 경우 정확한 쿼리는 `screen.getByRole("button", { name: "Sign up" });` 입니다. 이번에 액세스 가능한 이름은 버튼의 실제 text content입니다. 버튼에 `aria-label`을 추가하면 액세스 가능한 이름은 해당 `aria-label`의 text content가 됩니다.

최종적으로 업데이트된 테스트는 다음과 같습니다:

```jsx
describe("Form", () => {
  it("should submit correct form data", async () => {
    const user = userEvent.setup();
    const mockSave = jest.fn();
    render(<Form saveData={mockSave} />);

    await user.type(screen.getByRole("textbox", { name: "Name" }), "Test");
    await user.click(screen.getByRole("button", { name: "Sign up" }));

    expect(mockSave).toHaveBeenLastCalledWith({ ...defaultData, name: "Test" });
  });
});
```

> form component를 테스트하기 위해 React-Testing-Library를 사용하는 것에 관심이 있다면 이 문서가 도움이 될 수 있습니다: [Testing React Hook Form With React Testing Library.](https://claritydev.net/blog/testing-react-hook-form-with-react-testing-library)

### `*ByRole` vs `*ByLabelText` for input elements

입력 요소에 대해 `*ByRole` 쿼리를 사용하는 이유에 대해 궁금해할 수 있습니다. 그 목적은 input element를 해당 label과 연관지어 매칭하는 것입니다. 궁극적으로 같은 목표를 달성하고 문법도 더 간단한 `*ByLabelText` 쿼리를 사용하는 것이 더 쉽지 않을까요? 두 쿼리 사이에 큰 차이가 없어 보일 수 있지만, `*ByRole` 쿼리는 요소를 매칭할 때 [더 견고하며](https://testing-library.com/docs/queries/bylabeltext/#name), `<label>`에서 `aria-label`로 전환해도 여전히 작동합니다.

반면, 모든 유형의 input 요소가 기본 role을 가지는 것은 아닙니다. 예를 들어, password나 email input의 경우, `*ByLabelText` 쿼리를 사용해야 합니다. 따라서 두 방법 모두 장점이 있지만, 특정 사용 사례를 고려하고 상황에 가장 적합한 쿼리를 선택하는 것이 중요합니다.

### Use `userEvent` instead of `fireEvent`

우리는 이벤트를 시뮬레이션하기 위해 내장된 `fireEvent` 메서드를 사용하지 않고 대신 [userEvent](https://testing-library.com/docs/user-event/intro/) 메서드를 기본적으로 사용하는 것을 눈치챘을 것입니다. `fireEvent`가 많은 경우에 작동하지만, 이는 `dispatchEvent` API 위에 가벼운 래퍼로서 사용자 상호작용을 완전히 시뮬레이션하지 않습니다. 반면에, `userEvent`는 사용자가 브라우저에서 상호작용하는 것과 동일한 방식으로 DOM을 조작하여 더 신뢰할 수 있는 테스트 경험을 제공합니다. 또한, 이러한 접근 방식은 React Testing Library의 철학과도 더 잘 맞고, 문법이 더 명확합니다.

`testing-library/user-event` version 13부터는 이벤트를 시뮬레이션하기 전에 user event를 설정하는 것이 권장됩니다. 이후 버전에서는 이것이 필수가 되며, `userEvent`에서 직접 이벤트를 시뮬레이션하는 것이 더 이상 작동하지 않을 것입니다.

`userEvent`의 설정을 간소화하기 위해, 이벤트 설정과 컴포넌트 렌더링을 동시에 처리하는 유틸리티 함수를 추가할 수 있습니다.

```jsx
// setup userEvent
function setup(jsx) {
  return {
    user: userEvent.setup(),
    ...render(jsx),
  };
}

describe("Form", () => {
  it("should save correct data on submit", async () => {
    const mockSave = jest.fn();
    const { user } = setup(<Form saveData={mockSave} />);

    await user.type(screen.getByRole("textbox", { name: "Name" }), "Test");
    await user.click(screen.getByRole("button", { name: "Sign up" }));

    expect(mockSave).toHaveBeenLastCalledWith({ ...defaultData, name: "Test" });
  });
});
```

모든 `userEvent` 메서드는 비동기적이므로, 테스트도 약간 조정하여 비동기적으로 만들어야 합니다. 또한, `userEvent`는 별도의 패키지이므로 `npm i -D @testing-library/user-event` 명령어를 통해 설치해야 합니다.

### Simplify the `waitFor` queries with `findBy*`

종종 매칭하려는 요소가 초기 렌더링 시점에는 사용할 수 없는 경우가 있습니다. 예를 들어, API에서 항목을 가져온 후에 이를 표시할 때가 그러합니다. 이러한 상황에서는 쿼리를 실행하기 전에 컴포넌트가 모든 렌더링 사이클을 완료하도록 해야 합니다. 예를 들어, `ListPage` 컴포넌트를 수정하여 비동기적으로 항목 목록이 로드되기를 기다리도록 합시다:

```jsx
export const ListPage = () => {
  const [items, setItems] = useState([]);

  useEffect(() => {
    const loadItems = async () => {
      setTimeout(() => setItems(["Item 1", "Item 2"]), 100);
    };
    loadItems();
  }, []);

  if (!items.length) {
    return <div>Loading...</div>;
  }

  return (
    <div className='text-list__container'>
      <h1>List of items</h1>
      <ItemList items={items} />
    </div>
  );
};
```

현재 컴포넌트에 대한 테스트는 `screen.getByRole` 쿼리가 호출될 때 로딩 텍스트만 표시되기 때문에 더 이상 작동하지 않을 것입니다. 컴포넌트가 로딩을 완료할 때까지 기다리기 위해 `waitFor` helper를 사용할 수 있습니다. 다음은 수정된 테스트 코드입니다:

```jsx
import { waitFor } from "@testing-library/react";

//...

describe("ListPage", () => {
  it("renders without breaking", async () => {
    render(<ListPage />);

    await waitFor(() => {
      expect(
        screen.getByRole("heading", { name: "List of items" })
      ).toBeInTheDocument();
    });
  });
});
```

> Enzyme에서 React Testing Library로 테스트를 마이그레이션하는 것에 관심이 있다면, ["Enzyme vs React Testing Library: A Migration Guide"](https://claritydev.net/blog/enzyme-vs-react-testing-library-migration-guide)라는 아티클이 도움이 될 수 있습니다.

현재의 방식도 작동하지만, 비동기 동작이 내장된 쿼리 타입이 있습니다: `findBy*` queries. 이 쿼리들은 `waitFor`의 래퍼로서, 테스트를 더 읽기 쉽게 만듭니다:

```jsx
describe("ListPage", () => {
  it("renders without breaking", async () => {
    render(<ListPage />);
    expect(
      await screen.findByRole("heading", { name: "List of items" })
    ).toBeInTheDocument();
  });
});
```

주목할 점은 테스트 블록당 하나의 `await` 호출이 일반적으로 충분하다는 것입니다. 이는 모든 비동기 작업이 그 시점까지 해결되었기 때문입니다. 위 예제에서 추가로 `ItemList`에 4개의 item이 있는지 테스트하고자 한다면, 비동기 `findBy*` 쿼리를 사용할 필요 없이 `getBy*` 쿼리를 사용할 수 있습니다.

다음 예시는 원문에는 없지만 직접 추가한 예시임을 알려드립니다.

```jsx
describe("ListPage", () => {
  it("renders without breaking", async () => {
    render(<ListPage />);

    // Wait for the heading to be in the document
    expect(
      await screen.findByRole("heading", { name: "List of items" })
    ).toBeInTheDocument();

    // Check that the items are displayed
    const items = screen.getAllByRole("listitem");
    expect(items).toHaveLength(4);
  });
});
```

### Testing element's disappearance

이는 꽤 예외적인 경우이지만, 비동기 작업 후에 이전에 존재하던 요소가 DOM에서 제거되었는지 테스트하고자 할 때가 있습니다. React Testing Library는 이를 위해 유용한 helper 함수인 `waitForElementToBeRemoved`를 제공합니다. 예를 들어, `ListItem` 컴포넌트에서 리스트 헤더가 나타날 때까지 기다리는 대신, `Loading...` 텍스트가 제거될 때까지 기다리고 싶을 수 있습니다.

```jsx
it("renders without breaking", async () => {
  render(<ListPage />);
  await waitForElementToBeRemoved(() => screen.queryByText("Loading..."));
});
```

### Use React Testing Library Playground

어떤 요소에 대해 올바른 쿼리를 찾는 데 어려움을 겪는다면, [React Testing Library Playground](https://testing-playground.com/)가 큰 도움이 될 수 있습니다. 테스트 중인 컴포넌트의 HTML을 간단히 붙여넣으면 각 요소에 대해 적합한 쿼리에 대한 유용한 제안을 제공합니다. 이 도구는 복잡한 컴포넌트의 경우 특히 좋은데, 어떤 쿼리가 가장 적합한지 항상 명확하지 않을 때 유용합니다.

### Fixing the "not wrapped in act(...)" warnings

비동기 로직을 포함하는 컴포넌트와 작업할 때, 테스트를 실행하는 동안 경고 메시지를 접할 수 있습니다.

```text
Warning: An update to ComponentName inside a test was not wrapped in act(...).
```

이 경고는 테스트가 React가 컴포넌트를 업데이트하는 방식을 정확하게 시뮬레이션하지 못할 수 있음을 시사하며, 이는 테스트 결과에서 false positives 또는 negatives을 초래할 수 있습니다. 이 경고는 테스트가 React Testing Library에서 제공하는 `act` 함수 외부에서 상태 업데이트나 부작용을 트리거하여 React가 컴포넌트를 비동기적으로 업데이트할 때 발생합니다.

종종 이 경고의 원인은 비동기 작업 후 요소나 컴포넌트가 업데이트되는 경우에 `getBy*` 쿼리 대신 `findBy*` 쿼리를 사용하는 것입니다.

그러나 "act" 경고가 정당하며 테스트에서 false positives 또는 negatives을 수정하기 위해 해결해야 하는 경우도 있습니다. 그러한 경우 중 하나는 Jest 타이머와 작업할 때 발생합니다. 타이머가 있는 컴포넌트를 테스트할 때는 타이머의 흐름을 제어하고 불일치를 방지하기 위해 가짜 타이머(예: Jest의 `useFakeTimers`)를 사용합니다. 타이머를 실행하려면 `jest.runAllTimers();`를 사용할 수 있으며, 이는 또한 `act`로 감싸야 합니다.

```JS
jest.useFakeTimers();
// ... Set up the tests

act(() => {
  jest.runAllTimers();
});

// ... Do assertions

jest.useRealTimers();
```

이 경고는 React 18에서 더 자주 나타날 수 있는데, 이는 `useEffect`의 실행 방식에 일부 [변경](https://react.dev/blog/2022/03/08/react-18-upgrade-guide#other-breaking-changes)되었기 때문입니다. 이에 따라 더 많은 테스트를 `act`로 감싸야 할 필요가 있습니다.

"act" 경고를 해결하는 방법에 대한 보다 포괄적인 가이드는 다음 기사를 참조하십시오: [Fix the "not wrapped in act(...)" warning.](https://kentcdodds.com/blog/fix-the-not-wrapped-in-act-warning)

### Writing smoke tests

때때로 우리는 컴포넌트가 렌더링 중에 깨지지 않는지 확인하기 위해 기본적인 sanity 테스트를 원합니다. 다음 간단한 컴포넌트를 봅시다:

```JSX
export const ListPage = () => {
  return (
    <div className="text-list__container">
      <h1>List of items</h1>
      <ItemList />
    </div>
  );
};
```

다음과 같은 테스트를 통해 컴포넌트가 문제 없이 렌더링되는지 확인할 수 있습니다:

```jsx
import { render } from "@testing-library/react";
import React from "react";

import { ListPage } from "./ListPage";

describe("ListPage", () => {
  it("renders without breaking", () => {
    expect(() => render(<ListPage />)).not.toThrow();
  });
});
```

이 방식은 우리의 목적에 부합하지만, React Testing Library의 기능을 충분히 활용하지 못하고 있습니다. 대신, 다음과 같이 할 수 있습니다:

```jsx
import { render, screen } from "@testing-library/react";
import React from "react";

import { ListPage } from "./ListPage";

describe("ListPage", () => {
  it("renders without breaking", () => {
    render(<ListPage />);

    expect(
      screen.getByRole("heading", { name: "List of items" })
    ).toBeInTheDocument();
  });
});
```

비록 이것은 매우 단순화된 예시이지만, 이 작은 변경으로 인해 컴포넌트가 렌더링 중에 깨지지 않을 뿐만 아니라, 화면 판독기에서 제대로 접근할 수 있는 `List of items`라는 이름의 header 요소를 포함하고 있음을 테스트할 수 있습니다.

### Conclusion

이 아티클에서는 React Testing Library 테스트를 개선하기 위한 몇 가지 기법과 모범 사례를 탐구하고, 피해야 할 가장 일반적인 실수들을 나열했습니다. getByRole 쿼리를 사용함으로써 테스트가 좋은 커버리지를 제공할 뿐만 아니라 가치 있는 접근성 향상도 제공할 수 있음을 확인했습니다. 또한, fireEvent보다 userEvent 메서드를 사용하는 것의 이점과 최적의 사용을 위한 설정 방법을 배웠습니다. 마지막으로, findBy\* 쿼리와 waitForElementToBeRemoved를 사용하여 더 견고하고 신뢰할 수 있는 테스트를 작성하는 방법도 살펴보았습니다.

이러한 팁과 요령을 따르면 읽기 쉽고, 유지보수하기 쉬우며, [디버깅](https://claritydev.net/blog/beyond-console-log-debugging-techniques-javascript)하기 쉬운 더 나은 React 테스트를 작성할 수 있습니다. 테스트는 개발 과정의 필수적인 부분이며, 좋은 테스트를 작성하는 데 시간을 투자하는 것은 장기적으로 큰 이익을 가져올 수 있습니다. 이러한 기법과 모범 사례를 통해 우리의 React 애플리케이션이 철저하게 테스트되고 높은 품질을 유지할 수 있도록 보장할 수 있습니다.

### References and resources

[Beyond Console.log: Debugging Techniques In JavaScript](https://claritydev.net/blog/beyond-console-log-debugging-techniques-javascript)
[Creating Accessible Form Components with React](https://claritydev.net/blog/creating-accessible-form-components-with-react)
[Kent C. Dodds: Fix the "not wrapped in act(...)" warning](https://kentcdodds.com/blog/fix-the-not-wrapped-in-act-warning)
[MDN: The Label element](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/label)
[React Testing Library Playground](https://testing-playground.com/)
[React Testing Library documentation: ByLabelText query](https://testing-library.com/docs/queries/bylabeltext/#name)
[React Testing Library documentation: ByRole query](https://testing-library.com/docs/queries/byrole/)
[React Testing Library documentation](https://testing-library.com/docs/react-testing-library/intro/)
[React documentation: How to Upgrade to React 18](https://react.dev/blog/2022/03/08/react-18-upgrade-guide#other-breaking-changes)
[Testing Select Components with React Testing Library](https://claritydev.net/blog/testing-select-components-react-testing-library)
[TypeScript: Typing Form Events In React](https://claritydev.net/blog/typescript-typing-form-events-in-react)
[User Event documentation](https://testing-library.com/docs/user-event/intro/)
[What is an accessible name?](https://www.tpgi.com/what-is-an-accessible-name/)
[w3.org: HTML-ARIA](https://www.w3.org/TR/html-aria/#docconformance)
[w3.org: Placeholder text](https://www.w3.org/WAI/tutorials/forms/instructions/#placeholder-text)
