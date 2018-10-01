---
layout: post
title:  "What and How to Test with Jest and Enzyme. Full Instruction on React Components Testing"
date:   2018-09-28 09:41:18 +0300
categories: testing
---
![Cover image](/assets/images/cover.png)

Testing React components may be challenging for beginners and experienced developers who have already worked with tests. It may be interesting to compare your own approaches with the ones we use in our project. In order to cover the codebase, you have to know which components must be tested and which code exactly in component should be covered.

During reading, I’ll cover the next topics:
* Define the correct order of components’ testing based on project structure
* Find what to omit in test coverage (what not to test)
* Identify the necessity of Snapshot Testing
* Define what to test in the component and in which order
* Provide detailed custom code examples

The article requires that the reader already has knowledge about Jest and Enzyme setup. Information about installation and configuration can be easily found on the web or official websites.

Assume the following case: You need to cover the project codebase with tests, so what should you start with and what should you get at the end of testing? 100% test coverage? It is the indicator to which you should aspire, but in most situations you won’t get it. Why? Because you shouldn’t test all code. We will find out why and what should be left out of tests. Even more, 100% test coverage does not always ensure that the component is fully tested. As well, there is no guarantee it will notify you if something has been changed. Don’t strive for the percentages, avoid writing fake tests, and just try not to lose main component details.

## 📚 Table of Contents
* [Correct order of components’ testing](#correct-order-of-components-testing)
* [What should be omitted in test coverage](#what-should-be-omitted-in-test-coverage)
* [How to test with snapshots](#how-to-test-with-snapshots)
* [Main instructions for component testing](#main-instructions-for-component-testing)
* [Examples of Testing](#examples-of-testing)
  * [Testing Datepicker component](#testing-datepicker-component)
  * [Utility testing (valueToDate)](#utility-testing)
  * [Widgets testing (Spinner)](#widgets-testing)
  * [Modals testing (ModalWrapper and ModalTrigger)](#modals-testing)
    * [ModalWrapper testing](#modal-wrapper-testing)
    * [ModalTrigger testing](#modal-trigger-testing)
  * [HOC testing (BaseFieldHOC)](#hoc-testing)
  * [Form fields testing (BaseFieldLayout)](#fields-testing)

## <a name="correct-order-of-components-testing"></a>Defining the correct order of components’ testing based on project structure

Let’s discuss this question on the next part of project structure:

![Project structure image](/assets/images/project-structure.png)

I took `shared` directory because it is the most important; it consists of the components that are used in several different pages of the project. They are reusable and normally, they are small and not complex. If one or another component fails, it will cause failing in other places. That’s why we should be confident whether they have been written correctly. The structure of this directory is divided into several folders,  each containing components.

![Defining queue for Test Coverage image](/assets/images/testing-queue.png)

How to define the correct order of component testing in `shared` directory:
* Always follow the rule *from simple to complex*. Analyze each directory and define which components are `independent` - namely, their rendering doesn’t depend on the other components; they are self-completed and can be used separately as a single unit. From the structure above, it is `inputs` directory in `forms` folder. It contains input components to redux-forms, such as TextInput, SelectInput, CheckboxInput, DateInput, etc.
* Next, I need to define auxiliary components that are often used in `inputs` components, but should be tested apart from them. It is utils` directory. Components in this folder are not complicated, but very important. They are frequently reusable and help with repeated actions.
* The next step is to define which components can be used independently too. If any, take them for testing. From our structure, it is `widgets`, the little components with simple functionality. They will be the third item in the queue for test coverage.
* Further, analyze the rest of the directories and define more complex components, which can be used independently or in conjunction with other components. It is `modals` directory in our case; these components will be explained in detail below.
* The most complex for testing components are left in the end. They are `hoc` directory and `fields` from `forms` folder. How do you define which one should be tested first? I take the directory from which components have already been used in tested components. Thus, component from `hoc` directory was present in `widgets` component; that’s why I already know where and with which purpose this directory and its component are used.
* The last one is the `fields` folder; it contains components connected with redux-forms.

The final components order (based on our example) will look like this:

![Components order image](/assets/images/testing-order.png)

Following this order, you increase complexity of the tested components step by step; thus, when it comes to operating with the more complex components, you already know how the smallest ones behave. Don’t take for testing, for example, ‘array’ field, if you are not sure how to test ‘text’ field; don’t take components decorated with redux-form if you haven’t tested ‘form’ field itself. Be consistent in your choices, don’t take the first component that comes into your mind, and switch on logic. Of course, the structure of your project can differ; it can have other directory names or can have additional components, actions, and reducers, but the logic of defining the order for testing the components is the same.

**<a name="what-should-be-omitted-in-test-coverage"></a> Let’s define what should be omitted in test coverage:**
1. *Third-party libraries.* Don’t test functionality that is taken from another library; you are not responsible for that code. Skip it or imitate implementation if you need it to test your code.
2. *Constants.* The name speaks for itself. They are not changeable; it is a set of static code that is not intended to vary.
3. *Inline styles* (if you use them in component). In order to test inline styles, you need to duplicate object with styles in your test; if styles object changes, you must change it in test too. Don’t duplicate component’s code in tests; you will never keep in mind to change it in tests. Moreover, your colleague will never guess about duplication. In most cases, inline styles don’t change component’s behaviour; consequently, they shouldn’t be tested. There can be an exception in case your styles change dynamically.
4. *Things not related to the tested component.* Skip covering with tests components that were imported in the tested component; be careful if it is wrapped in another one. Don’t test wrapper, just analyze and test them separately.

So how do you actually write tests? I combine two testing approaches:
* Snapshot Testing
* Component logic testing

Snapshot Testing is a useful testing tool in case you want to be sure user interface hasn’t changed. When facing this testing tool for the first time, questions arise concerning organization and managing snapshots. The principle of work is very simple, but unfortunately it has not been fully described anywhere; on the official website jestjs.io, description of Snapshot Testing work is very poor.

## <a name="how-to-test-with-snapshots"></a> How to test with snapshots
**Step 1**. Write test for the component and in the expect block, use `.toMatchSnapshot()` method that creates `Snapshot` itself.

```
it('render correctly text component', () => {
    const TextInputComponent = renderer.create(<TextInput />).toJSON();
    expect(TextInputComponent).toMatchSnapshot();
});
```

**Step 2**. When you run test for the first time on the one level, along with the test, there will be a created directory named `__snapshots__` with the autogenerated file inside with the extension `.snap`.

Snapshot looks like:

```
// Jest Snapshot v1, https://goo.gl/fbAQLP

exports[`Render TextInput correctly component 1`] = `
<input
  className="input-custom"
  disabled={undefined}
  id={undefined}
  name={undefined}
  onBlur={undefined}
  onChange={[Function]}
  pattern={undefined}
  placeholder={undefined}
  readOnly={false}
  required={undefined}
  type="text"
  value={undefined}
/>
`;
```

**Step 3**. Push snapshot into the repository and store it along with the test.

If component has been changed, you need just update snapshot with `—updateSnapshot` flag or using shot form `u` flag.

**Snapshot is created; how does it work?**

Let us consider two cases:

1. The component has changed

    * Run tests

    * New snapshot is created, it compares with the auto generated snapshot stored in the directory `__snapshots__`

    * Tests failed because snapshot is different

    ![Tests failed animation](/assets/images/fail.gif)

2. The component has not changed

    * Run tests

    * New snapshot is created, it compares with the auto generated snapshot stored in the directory `__snapshots__`

    * Tests passed because snapshot is identical

    ![Tests succeeded animation](/assets/images/success.gif)

Everything is fine when I test a small component without logic, just UI rendering, but as practice shows, there are no such components on real projects. If they are, they are in a small amount.

**Is there enough snapshot for full component testing?**

The answer is “No”. I stick to the approach to combine Snapshot Testing with manually written tests for component logic. The reason is in the component’s states, props, and conditions inside. One snapshot can be generated with one current state or condition of the component. What do you do with the others? Create 5-10 snapshots and store them along with the test? I would say it is not a right way, so don’t practice this technique. Use snapshots for testing UI and manually written tests for testing functionality.

## <a name="main-instructions-for-component-testing"></a>Main instructions for component testing

1. One component should have only one snapshot. If one snapshot fails, most likely the others will fail too, so do not create and store a bunch of unnecessary snapshots clogging the space and confusing developers who will read your tests after you. Of course, there are exceptions when you need to test the behavior of a component in two states; for example, in the state of the component before opening the pop-up and after opening. However, even such variant can always be replaced by this one: the first test stores default state of the component without popup in snapshot, and the second test simulates event and checks the presence of a particular class. In this way, you can easily bypass the creation of several snapshots.

2. Testing props: As a rule, I divide the testing of the props into two tests:

    * Firstly, check the render of default prop values; when the component is rendered, I expect a value to be equal from `defaultProps` in case this prop has `defaultProps`.

    * Secondly, check the custom value of the prop; I set my own value and expect it to be received after the render of the component.

3. Testing data types: In order to test what type of data comes in the props or what kind of data is obtained after certain actions, I use the special library jest-extended (Additional Jest matchers), which has an extended set of matches that are absent in the Jest. With this library, testing of data types is much easier and more enjoyable. Testing proptypes is a contradictory question. Some developers can argue against proptypes testing because it is a third-party package and shouldn’t be tested, but I insist on testing components’ proptypes because I don’t test the package functionality itself; I just ensure the proptypes are correct. Data type is a very important programming part and shouldn’t be skipped.

4. Event testing: After creating a snapshot and covering props with tests, you can be sure in correct rendering of the component, but this is not enough for full coverage in case you have events in the component. You can check event in several ways; the most widely used are:

    * mock event => simulate it => expect event was called

    * mock event => simulate event with params => expect event was called with passed params

    * pass necessary props => render component => simulate event => expect a certain behavior on called event

5. Testing conditions: Very often you can have conditions for the output of a particular class, rendering a certain section of the code, transferring the required props, and so on. Do not forget about this because with default values, only one branch will pass the test, while the second one will remain untested. In  complex components with calculations and lots of conditions, you can miss some branches. To make sure all parts of the code are covered by tests, use test coverage tool and visually check which branches are covered and which are not.

6. States’ testing: To check state, in most cases, it is necessary to write two tests:
    * The first one checks current state.
    * The second one checks state after calling event. Render component => call function directly in the test => check how state has changed. To call function of the component, you need to get an instance of the component and only then call its methods (example is shown in [next test](https://github.com/ned-alyona/testing-jest-enzyme/blob/master/shared/modals/__tests__/ModalTrigger.test.js#L30)).

After you walk through this list of instructions, your component will be covered from 90 to 100%. I leave 10% for special cases that were not described in the article, but can occur in the code.

## <a name="examples-of-testing"></a>Examples of Testing

Let’s move to examples and cover components with tests under described above structure step by step.

**<a name="testing-datepicker-component"></a>1. Testing of a component from forms/inputs.**
Take one component from *forms/inputs* directory; let it be DateInput.js, the component for datepicker field.

**Code listing for tested component:** [DateInput.js](https://github.com/ned-alyona/testing-jest-enzyme/blob/master/shared/forms/inputs/DateInput.js)

**Looks like:**

![Datepicker animation](/assets/images/datepicker.gif)

DateInput component uses library react-datepicker, with two utilities: valueToDate (converts value to date) and dateToValue is vice versa, moment package for manipulating with date and PropTypes for checking React props.

According to the component code, we can see the list of default props with the help of which the component should be rendered:

```
const defaultProps = {
    inputClassName: 'input-custom',
    monthsShown: 1,
    dateFormat: 'DD.MM.YYYY',
    showMonthYearsDropdowns: false,
    minDate: moment()
};
```

All props are appropriate for creating snapshot, except one `minDate: moment()`, moment() will give us current date each time we run test and snapshot will fail because it stores outdated date. Solution is to mock this value:

```
const defaultProps = {
    minDate: moment(0)
}
```

minDate prop I need in each rendered component; to avoid props duplication, I create HOC which receives defaultProps and returns pretty component:

```
import TestDateInput from '../DateInput';

const DateInput = (props) =>
    <TestDateInput
        {...defaultProps}
        {...props}
    />;
```

Don’t forget about `moment-timezone`, especially if your tests will be run by developers from another country in a different time zone. They will receive mocked value, but with time zone shift. Solution is to set default time zone:

```
const moment = require.requireActual('moment-timezone').tz.setDefault('America/Los_Angeles')
```

Now date input component is ready for testing:

1. Create snapshot first:

    ```
    it('render correctly date component', () => {
        const DateInputComponent = renderer.create(<DateInput />).toJSON();
        expect(DateInputComponent).toMatchSnapshot();
    });
    ```


2. Testing props:

    Look through props and find important; first prop to test is showMonthYearsDropdowns, if it set to true, the dropdown for month and years is shown:

    ```
    it('check month and years dropdowns displayed', () => {
        const props = {
                showMonthYearsDropdowns: true
            },
            DateInputComponent = mount(<DateInput {...props} />).find('.datepicker');
        expect(DateInputComponent.hasClass('react-datepicker-hide-month')).toEqual(true);
    });
    ```

    Test null prop value; this check is required to ensure the component is rendered without defined value:

    ```
    it('render date input correctly with null value', () => {
        const props = {
                value: null
            },
            DateInputComponent = mount(<DateInput {...props} />);
        expect((DateInputComponent).prop('value')).toEqual(null);
    });
    ```

3. Test proptypes for value, date expected to be string:

    ```
    it('check the type of value', () => {
        const props = {
                value: '10.03.2018'
            },
            DateInputComponent = mount(<DateInput {...props} />);
        expect(DateInputComponent.prop('value')).toBeString();
    });
    ```

4. Test events:

    4.1. Check the onChange event, for that mock onChange callback => render date input component => then simulate change event with new target value => and finally check that onChange event have been called with new value.

    ```
    it('check the onChange callback', () => {
        const onChange = jest.fn(),
            props = {
                value: '20.01.2018',
                onChange
            },
            DateInputComponent = mount(<DateInput {...props} />).find('input');
        DateInputComponent.simulate('change', { target: {value: moment('2018-01-22')} });
        expect(onChange).toHaveBeenCalledWith('22.01.2018');
    });
    ```

    4.2. Ensure datepicker popup opens after click on date input, for that find date input => simulate click event => and expect popup with class `.react-datepicker` is present.

    ```
    it('check DatePicker popup open', () => {
        const DateComponent = mount(<DateInput />),
            dateInput = DateComponent.find("input[type='text']");
        dateInput.simulate('click');
        expect(DateComponent.find('.react-datepicker')).toHaveLength(1);
    });
    ```

*Full tests listing:* [DateInput.test.js](https://github.com/ned-alyona/testing-jest-enzyme/blob/master/shared/forms/inputs/__tests__/DateInput.test.js)

**<a name="utility-testing"></a>2. Utility testing:**

**Code listing for tested utility:** [valueToDate.js](https://github.com/ned-alyona/testing-jest-enzyme/blob/master/shared/utils/valueToDate.js)

Purpose of this utility is transforming value to date with custom format.

First of all, let’s analyze given utility and define the main cases for testing:

1. According to the purpose of this utility, it transforms value, so we need to check this value:
    * In case value is not defined: we need to be sure that utility will not return exception (error).
    * In case value defined: we need to check that utility returns moment date.

2. The returned value should belong to the moment class; that’s why it should be instance of moment.
3. Second argument is dateFormat; set it as constant before tests. That’s why it will be passed in each test and return value according to date format. Should we test dateFormat separately? I suppose no. This argument is optional; if we don’t set dateFormat, the utility won’t break, and it’ll just return date in default format; it is a moment job, we shouldn’t test third-party libraries.

As I mentioned before, we shouldn’t forget about moment-timezone; it is a very important point, especially for developers from different time zones.

**Let’s code:**

1. Write test for first case; when we don’t have value, it is empty.

    ```
    const format = 'DD.MM.YYYY';

    it('render valueToDate utility with empty value', () => {
        const value = valueToDate('', format);
        expect(value).toEqual(null);
    });
    ```

2. Check if value is defined.

    ```
    const date = '21.11.2015',
          format = 'DD.MM.YYYY';

    it('render valueToDate utility with defined value', () => {
        const value = valueToDate(date, format);
        expect(value).toEqual(moment(date, format));
    });
    ```

3. Check the value’s belonging to the moment class.

    ```
    const date = '21.11.2015',
        format = 'DD.MM.YYYY';

    it('check value is instanceof moment', () => {
        const value = valueToDate(date, format);
        expect(value instanceof moment).toBeTruthy();
    });
    ```

*Full tests listing:* [valueToDate.test.js](https://github.com/ned-alyona/testing-jest-enzyme/blob/master/shared/utils/__tests__/valueToDate.test.js)

**<a name="widgets-testing"></a> 3. Widgets testing**

For widgets testing, I took spinner component.

**Code listing for tested widget:** [Spinner.js](https://github.com/ned-alyona/testing-jest-enzyme/blob/master/shared/widgets/Spinner.js)

**Looks like:**

![Spinner animation](/assets/images/spinner.gif)

Spinner is not required in explanation, as almost all web resources have this component.
So go to write tests:

1. First step - create snapshot:

    ```
    it('render correctly Spinner component', () => {
       const SpinnerComponent = mount(<Spinner />);
       expect(SpinnerComponent).toMatchSnapshot();
    });
    ```

2. Testing props:

    2.1 Default prop title, check if it renders correctly.

    ```
    it('check prop title by default', () => {
        const SpinnerComponent = mount(<Spinner />);
        expect(SpinnerComponent.find('p').text()).toEqual('Please wait');
    });
    ```

    2.2 Check custom prop title; I need to check that it returns correctly defined prop. Take a look at the code, the title wrapped in rawMarkup util, and outputs with the help of dangerouslySetInnerHTML property.

    Code listing for rawMarkup util:

    ```
    export default function rawMarkup(template) {
        return {__html: template};
    }
    ```

    Do we need to include tests for rawMarkup in spinner component? No, it is a separate utility and it should be tested apart from the spinner. We don’t care how it works; we just need to know that title prop returns correct result.

    Clarification: The reason of using dangerouslySetInnerHTML property is the following. Our site is multilingual, for which translations marketing team is responsible. They can translate it simply with combination of words or even decorate with the html tags, like `<strong>`, `<i>`, `<s>` or even slice text with the lists `<ol>`, `<ul>`; we don’t know for sure how they translate and decorate the text. We just need to correctly render all this stuff.

    I combined two main test cases in one test:
    * return correct custom prop title

    * render correctly prop title with html tags

    ```
    it('check prop title with html tags', () => {
        const props = {
                title: '<b>Please wait</b>'
            },
            SpinnerComponent = mount(<Spinner {...props} />);
        expect(SpinnerComponent.find('p').text()).toEqual('Please wait');
    });
    ```

    Take next prop subTitle; it is optional and that’s why it doesn’t have default prop, so skip step with default props and test custom props:

    * Check that text in subTitle prop renders correctly:

    ```
    const props = {
            subTitle: 'left 1 minute'
        },
        SpinnerComponent = mount(<Spinner {...props} />);

    it('render correct text', () => {
        expect(SpinnerComponent.find('p').at(1).text()).toEqual(props.subTitle);
    });
    ```

    * We know that subTitle is optional; that’s why we need to check whether it is not rendered with default props, according to the slicing markup. Just check the number of tags `<p>`:

    ```
    it('check subTitle is not rendered', () => {
        const SpinnerComponent = mount(<Spinner />);
        expect(SpinnerComponent.find('p').length).toEqual(1);
    });
    ```

3. Testing prop types:
   * For title prop expected to be string:

    ```
    it('check prop type for title is string', () => {
       const props = {
               title: 'Wait'
           },
           SpinnerComponent = mount(<Spinner {...props} />);
       expect(SpinnerComponent.find('p').text()).toBeString();
    });
    ```

   * For subTitle prop also expected to be string:

    ```
    const props = {
           subTitle: 'left 1 minute'
       },
       SpinnerComponent = mount(<Spinner {...props} />);

    it('type for subTitle is string', () => {
       expect(SpinnerComponent.find('p').at(1).text()).toBeString();
    });
    ```

*Full tests listing:* [Spinner.test.js](https://github.com/ned-alyona/testing-jest-enzyme/blob/master/shared/widgets/__tests__/Spinner.test.js)

**<a name="modals-testing"></a>4. Modals testing (ModalWrapper.js and ModalTrigger.js)**

**Looks like:**

![Spinner animation](/assets/images/modal.gif)

**How to test modals:**

First of all, I want to explain how modals are organized on our project. We have two components: ModalWrapper.js and ModalTrigger.js.

*ModalWrapper* is responsible for popup layout. It contains modal container, button ‘close’, modal title and body.

*ModalTrigger* is responsible for modal handling. It includes ModalWrapper layout and contains events for modal’s layout control (open, close actions).

I overview each component separately:

**<a name="modal-wrapper-testing"></a>1. Code listing for tested component:** [ModalWrapper.js](https://github.com/ned-alyona/testing-jest-enzyme/blob/master/shared/modals/ModalWrapper.js)

**Let’s code:**

   1.1 ModalWrapper receives component and renders it inside. First of all, check that ModalWrapper won’t fail without component. Create snapshot with default props:

```
it('without component', () => {
   const ModalWrapperComponent = shallow(<ModalWrapper />);
   expect(ModalWrapperComponent).toMatchSnapshot();
});
```

   1.2 Next step is to simulate its actual condition with component rendering passed through props:

```
it('with component', () => {
  const props = {
          component: () => {}
       },
       ModalWrapperComponent = shallow(<ModalWrapper {...props} />);
   expect(ModalWrapperComponent).toMatchSnapshot();
});
```

   1.3 Testing props:
   * Receiving custom class name prop:

```
it('render correct class name', () => {
  const props = {
          modalClassName: 'custom-class-name'
      },
      ModalWrapperComponent = shallow(<ModalWrapper {...props} />).find('Modal');
      expect(ModalWrapperComponent.hasClass('custom-class-name')).toEqual(true);
});
```

   * Receiving custom title prop:

```
it('render correct title', () => {
  const props = {
         title: 'Modal Title'
     },
     ModalWrapperComponent = shallow(<ModalWrapper {...props} />).find('ModalTitle');
  expect(ModalWrapperComponent.props().children).toEqual('Modal Title');
});
```

   * Receiving correct show prop:

```
it('check prop value', () => {
  const props = {
         show: true
     },
     ModalWrapperComponent = shallow(<ModalWrapper {...props} />).find('Modal');
  expect(ModalWrapperComponent.props().show).toEqual(true);
});
```

   1.4 Testing proptypes:
   * for show prop

```
it('check prop type', () => {
  const props = {
         show: true
      },
      ModalWrapperComponent = shallow(<ModalWrapper {...props} />).find('Modal');
  expect(ModalWrapperComponent.props().show).toBeBoolean();
});
```

   * for onHide prop

```
it('render correct onHide prop type', () => {
  const props = {
          onHide: () => {}
      },
      ModalWrapperComponent = shallow(<ModalWrapper {...props} />).find('Modal');
  expect(ModalWrapperComponent.props().onHide).toBeFunction();
});
```

   * for component prop

```
it('render correct component prop type', () => {
 const props = {
         component: () => {}
     },
     ModalWrapperComponent = mount(<ModalWrapper {...props} />);
 expect(ModalWrapperComponent.props().component).toBeFunction();
});
```

*Full tests listing:* [ModalWrapper.test.js](https://github.com/ned-alyona/testing-jest-enzyme/blob/master/shared/modals/__tests__/ModalWrapper.test.js)

**<a name="modal-trigger-testing"></a>2. Code listing for tested component:** [ModalTrigger.js](https://github.com/ned-alyona/testing-jest-enzyme/blob/master/shared/modals/ModalTrigger.js)

The modal wrapper has covered with test; the second part is to cover modal trigger component.
Component overview: it is based on the **state `toggled`** that indicates visibility of ModalWrapper. If **`toggled: false`**, the popup is hidden, else visible. Function **open()** opens popup on child element; click event and function **close()** hides popup on button rendered in ModalWrapper.

   2.1 Snapshot creation:

```
it('render ModalTrigger component correctly', () => {
   const ModalTriggerComponent = shallow(<ModalTrigger><div /></ModalTrigger>);
   expect(ModalTriggerComponent).toMatchSnapshot();
});
```

   Should we test ModalTrigger with component prop rendering? No; because `component` will be rendered inside ModalWrapper component, it does not depend on the tested component. It was already covered with tests in the ModalWrapper tests.

   2.2 Testing props. We have one prop `children` and we want to be sure that we have only one child.

```
it('ensure to have only one child (control element)', () => {
   expect(ModalTriggerComponent.findWhere(node => node.key() === 'modal-control').length).toEqual(1);
});
```

   2.3 Testing proptypes. Children prop should be object, check this in the next test:

```
const ModalTriggerComponent = mount(<ModalTrigger><div /></ModalTrigger>);

it('check children prop type', () => {
    expect(ModalTriggerComponent.props().children).toBeObject();
});
```

   2.4 Important part of ModalTrigger component is to check states.

   We have two states:
   * Popup is opened. To know that modal is opened, we need to check its state. For this, call open function from instance of the component and expect that *`toggled`* in state should be true.

```
it('check the modal is opened', () => {
   const event = {
       preventDefault: () => {},
       stopPropagation: () => {}
   };
   ModalTriggerComponent.instance().open(event);
   expect(ModalTriggerComponent.state().toggled).toBeTruthy();
});
```

   * Popup is closed, is tested vice versa, *`toggled`* in state should be false.

```
it('check the modal is closed', () => {
  ModalTriggerComponent.instance().close();
  expect(ModalTriggerComponent.state().toggled).toBeFalsy();
});
```

*Full tests listing:* [ModalTrigger.test.js](https://github.com/ned-alyona/testing-jest-enzyme/blob/master/shared/modals/__tests__/ModalTrigger.test.js)

Now modals are fully tested. One piece of advice for testing the components that are dependent on each other: look through the components first and write test plan, define what you need to test in each component, check test cases for each component, and be sure you don’t repeat the same test case in both components. Carefully analyze possible and optimal variants for test coverage.

**<a name="hoc-testing"></a>5. HOC testing (Higher-Order Component)**

The last two parts (HOC and form’s fields testing) are interconnected. I would like to share with you how to test field layout with its HOC.

Explanation of what is the BaseFieldLayout, why we need this component, and where we use it:
* BaseFieldLayout.js is the wrapper for form input components like TextInput, CheckboxInput, DateInput, SelectInput, etc. Their names end with the `-Input` because we use redux-form package and these components are the input components to redux-form logic.
* We need BaseFieldLayout for creating layout for form fields components, that is rendering label, tooltips, prefixes (currency, square meter abbreviations, etc.), icons, errors …
* We use it in BaseFieldHOC.js for wrapping inputComponent in field layout and connect it with the redux-form with the help of `<Field/>` component.

**Code listing for tested component:** [BaseFieldHOC.js](https://github.com/ned-alyona/testing-jest-enzyme/blob/master/shared/hoc/BaseFieldHOC.js)

It is a HOC which receives form input component and returns component, connected with redux-form.

Analyze HOC:
* This component receives only one prop, `component`. First of all, I need to create this component and wrap it in the BaseFieldHOC.
* Next, I need to decorate wrapped HOC with redux-form in order to get field connected with redux-form.
* Render this field inside React Redux `<Provider>` component to make the store available to tested component.
To mock store, just do:

```
const store = createStore(() => ({}));
```

Now, before each test, I need to do the next:

```
let BaseFieldHOCComponent;

beforeEach(() => {
    const TextInput = () => { return 'text input'; },
        BaseFieldHOCWrapper = BaseFieldHOC(TextInput),
        TextField = reduxForm({ form: 'testForm' })(BaseFieldHOCWrapper);
    BaseFieldHOCComponent = renderer.create(
        <Provider store={store}>
            <TextField name="text-input" />
        </Provider>
    ).toJSON();
});
```

After that, the component is ready for testing:

1. Create snapshot:

    ```
    it('render correctly component', () => {
        expect(BaseFieldHOCComponent).toMatchSnapshot();
    });
    ```

2. Ensure that input component is wrapped in BaseFieldLayout after rendering:

    ```
    it('check input component is wrapped in BaseFieldLayout', () => {
        expect(BaseFieldHOCComponent.props.className).toEqual('form-group');
    });
    ```

That’s all, the HOC is covered. The most complicated part in testing connected with redux-form components is to make preparation of the field (decorate with redux form and setup store); the rest is easy, just follow the instructions and nothing else.

*Full tests listing:* [BaseFieldHOC.test.js](https://github.com/ned-alyona/testing-jest-enzyme/blob/master/shared/hoc/__tests__/BaseFieldHOC.test.js)

**<a name="fields-testing"></a>6. Forms/fields testing**

Field HOC has covered with tests and we can move to BaseFieldLayout component.

**Code listing for tested component:** [BaseFieldLayout.js](https://github.com/ned-alyona/testing-jest-enzyme/blob/master/shared/forms/fields/BaseFieldLayout.js)

Let’s code BaseFieldLayout.js; write tests according to instructions above:

1. First of all, create snapshot. This component will not be rendered without defaultProps:
    * inputComponent
    * The props provided by redux-form: input and meta objects. Input with property name and meta with properties error and touched:

    ```
    const defaultProps = {
        meta: {
            touched: null,
            error: null
        },
        input: {
            name: 'field-name'
        },
        inputComponent: () => { return 'test case'; }
    }
    ```

    To use defaultProps in each tested wrapper, do the following:

    ```
    import TestBaseFieldLayout from '../BaseFieldLayout';

    const BaseFieldLayout = (props) => <TestBaseFieldLayout {...defaultProps} {...props} />;
    ```

    Now we are ready to create snapshot:

    ```
    it('render correctly BaseFieldLayout component', () => {
        const BaseFieldLayoutComponent = renderer.create(<BaseFieldLayout />).toJSON();
        expect(BaseFieldLayoutComponent).toMatchSnapshot();
    });
    ```

2. Testing props:

    This component has many props. I will show examples of several ones; the rest are tested by analogy.

    * Ensure `icon` prop is rendered correctly

    ```
    it('render correctly icon prop', () => {
       const props = {
               icon: <span className="icon-exclamation" />
           },
           BaseFieldLayoutComponent = mount(<BaseFieldLayout {...props} />);
           expect(BaseFieldLayoutComponent.find('span').hasClass('icon-exclamation')).toBeTruthy();
    });
    ```

    * Ensure tooltip content renders next to the label

    ```
    const props = {
           labelTooltipContent: 'tooltip for label'
       },
       BaseFieldLayoutComponent = mount(<BaseFieldLayout {...props} />);

    it('check prop is rendered', () => {
      expect(BaseFieldLayoutComponent.find('span').hasClass('tooltip-icon')).toBeTruthy();
    });
    ```

    * Testing `fieldLink` prop
    * Ensure fieldLink is null by default

    ```
    it('check prop is null by default', () => {
        const BaseFieldLayoutComponent = shallow(<BaseFieldLayout />);
        expect(BaseFieldLayoutComponent.props().fieldLink).toBe(null);
    });
    ```

    * Ensure fieldLink renders correctly with custom value

    ```
    describe('render correctly fieldLink', () => {
        const props = {
            fieldLink: <a href="#" />
        };

        it('check prop is rendered with defined link', () => {
            const BaseFieldLayoutComponent = mount(<BaseFieldLayout {...props} />);
            expect(BaseFieldLayoutComponent.contains(props.fieldLink)).toBeTruthy();
        });
    });
    ```

3. Testing errors:

    ```
    it('check if field has error', () => {
        const props = {
                meta: {
                    touched: true,
                    error: 'This field is required'
                }
            },
            BaseFieldLayoutComponent = mount(<BaseFieldLayout {...props} />);
        expect(BaseFieldLayoutComponent.find('.error')).toHaveLength(1);
    });
    ```

*Full tests listing:* [BaseFieldLayout.test.js](https://github.com/ned-alyona/testing-jest-enzyme/blob/master/shared/forms/fields/__tests__/BaseFieldLayout.test.js)

## Bottom Line
Now you have full guidance on how to perform full coverage testing of components based on project structure. From my own experience, I tried to explain what is necessary to test, in which order, and what you can omit in test coverage. Also, I demonstrated examples of several testing components and spotted the sequence of codebase coverage. I hope that you will find this article useful and will share your response. Thank you for reading.

Originally published on [Django Stars blog](https://djangostars.com/blog/what-and-how-to-test-with-enzyme-and-jest-full-instruction-on-react-component-testing/).
