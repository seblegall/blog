---
title: "Form validation with React using Formik and Redux"
description: "How to do sync (JS) and async (calling the api) validation in React using Formik and redux"
date:   2018-03-27T13:00:00+02:00
author: Sébastien Le Gall
categories: [React, Formik, Redux, MaterialUI]
---

Form handling is a huge web development topic. There are almost only 2 ways to deal with form, depending on the lib / the framework you choose to use and depending on the side you are trying to handle your form (client vs server) :

* The ones that include form handling a component. This component usually does a lot of magic and force you to think your form the way it is going to be handle by the component.
* The ones that let you do all the work. And then, you feel like it's a non-sens having to manage your form at such a low level by yourself

## Simple use case in React

As long as you work with simple form in react, things stay pretty simple. You basically just have to worry about your `onSubmit` method that will call your API in order to POST form data.

Here is a basic example of a form posting a *snippet* (a title and a text) using React, Material UI and Redux (After posting the data, the method `addSnippet` dispatch an event in order to refresh another component)

```js
import React from 'react';
import Dialog from 'material-ui/Dialog';
import FlatButton from 'material-ui/FlatButton';
import TextField from 'material-ui/TextField';
import {addSnippet} from "../../../actions/snippets";
import {connect} from "react-redux";
import { bindActionCreators } from 'redux';


class AddSnippet extends React.Component {

    constructor(props) {
        super(props);
        this.handleSubmit = this.handleSubmit.bind(this);
    }

    handleSubmit = (event) => {
        event.preventDefault();
        const form = new FormData(event.target);

        const data = {
            "title": form.get("title"),
            "snippet": form.get("snippet")
        };

        this.props.addSnippet(data);
    };

    render() {
        const actions = [
            <FlatButton
                label="Cancel"
                primary={true}
                onClick={this.handleClose}
                key="cancel"
            />,
            <FlatButton
                type="submit"
                label="Submit"
                primary={true}
                keyboardFocused={true}
                key="submit"
            />,
        ];

        return (
            <div>
                <Dialog
                    title="Adding a snippet"
                    modal={false}
                    open={this.state.open}
                    onRequestClose={this.handleClose}>
                    <form method="POST" onSubmit={this.handleSubmit}>
                        <TextField
                            hintText="My awesome snippet"
                            floatingLabelText="Title"
                            name="title"
                        /><br />
                        <TextField
                            hintText="<?php echo 'Hello World'; ?>"
                            floatingLabelText="Snippet"
                            multiLine={true}
                            rows={5}
                            rowsMax={10}
                            fullWidth={true}
                            name="snippet"
                        /><br />
                        <div style={{ textAlign: 'right', padding: 8, margin: '24px -24px -24px -24px' }}>
                            {actions}
                        </div>
                    </form>
                </Dialog>
            </div>
        );
    }
}

const mapDispatchToProps = dispatch => (bindActionCreators({
    addSnippet
}, dispatch));

export default connect(
    null,
    mapDispatchToProps
)(AddSnippet)
```

But then... you'll probably want to do some kind of validation.

The first step is to do client side validation such as : if a field is required, you'll surely want to check if the field is not empty before calling your API. In this example we would have to do that manually. Initializing a state with the fields and then check the state before calling the API could be a good idea. But still... It takes time and can be a source of errors / bug.

The second step is to do server side validation. Aka : calling your API and check for status code > 299. Then, get the returned payload hopping there is a common schema for validation errors and map the errors sent by the API to your fields. This part is a little bit tricky and also a source of errors / bug.

## Form validation made simple with Formik

[Formik](https://github.com/jaredpalmer/formik) is a react lib that helps you do the basics with forms without introducing any magic. You can see it as a collection of tools you may need when handling forms such as : `handleSubmit`, `handleChange`, `handleBlur`, `setErrors`, ets.

You will find a commented example bellow. It took me time to understand how to properly handle errors coming from the API, so this example also document the API call. The idea is to understand the whole thing here.

This example use Formik, Redux, [Yup](https://github.com/jquense/yup) for client side validation, and [Material UI](http://www.material-ui.com/) for design. The form is located in a `Dialog` ([Dialog](http://www.material-ui.com/#/components/dialog)) that opens when the user click on the `FloatingActionButton` ([FloatingActionButton](http://www.material-ui.com/#/components/floating-action-button))

Keep in mind that, in case of errors, the API is supposed to return a payload structured like :

```json
{
    "errors": [
        {
            "field": "title",
            "error": "This field is required"
        }
    ]
}
```

My component file : `AddSnippet`
```js
import React from 'react';
import Dialog from 'material-ui/Dialog';
import FlatButton from 'material-ui/FlatButton';
import FloatingActionButton from 'material-ui/FloatingActionButton';
import ContentAdd from 'material-ui/svg-icons/content/add';
import TextField from 'material-ui/TextField';
import {addSnippet} from "../../../actions/snippets";
import {connect} from "react-redux";
import { bindActionCreators } from 'redux';
import { Formik } from 'formik';
import Yup from 'yup'


class AddSnippet extends React.Component {

    constructor(props) {
        super(props);
        this.state = {
            open: false,
        };
        this.handleClose = this.handleClose.bind(this);
    }

    handleOpen = () => {
        this.setState({open: true});
    };

    handleClose = () => {
        this.setState({open: false});
    };

    render() {
        return (
            <div>
                <FloatingActionButton secondary={true} onClick={this.handleOpen} style={{
                    margin: 0,
                    top: 'auto',
                    right: 20,
                    bottom: 20,
                    left: 'auto',
                    position: 'fixed',
                }}>
                    <ContentAdd />
                </FloatingActionButton>
                <Dialog
                    title="Adding a snippet"
                    modal={false}
                    open={this.state.open}
                    onRequestClose={this.handleClose}>
                    <Formik
                        initialValues={{title: '', snippet: ''}}
                        onSubmit={async (values, {setFieldError}) => {
                            try {
                                await this.props.addSnippet(values); // Call the api
                                this.handleClose();
                            } catch (errors) { // Catch status code > 299
                                errors.forEach( err => {
                                    setFieldError(err.field, err.error); // Map errors to fields
                                });
                            }
                        }}
                        validationSchema={Yup.object().shape({
                            title: Yup.string() //Client side validation for field "title"
                                .min(3, 'Title must be at least 3 characters long.')
                                .required('Title is required.'),
                            snippet: Yup.string() //Client side validation for field "snippet"
                                .min(3, 'Snippet must be at least 3 characters long.')
                                .required("Snippet is required"),
                        })}
                        component={ this.form }
                    />
                </Dialog>
            </div>
        )
    }

    form = ({handleSubmit, handleChange, handleBlur, values, errors}) => {
        return (
            <form method="POST" onSubmit={handleSubmit}>
                <TextField
                    hintText="My awesome snippet"
                    floatingLabelText="Title"
                    name="title"
                    onChange={handleChange} //By default client side validation is done onChange
                    onBlur={handleBlur} //By default client side validation is also done onBlur
                    value={values.title}
                    errorText={errors.title} //Error display
                /><br />
                <TextField
                    hintText="<?php echo 'Hello World'; ?>"
                    floatingLabelText="Snippet"
                    multiLine={true}
                    rows={5}
                    rowsMax={10}
                    fullWidth={true}
                    name="snippet"
                    onChange={handleChange}
                    onBlur={handleBlur}
                    value={values.snippet}
                    errorText={errors.snippet} //Error display
                /><br />
                <div style={{ textAlign: 'right', padding: 8, margin: '24px -24px -24px -24px' }}>
                    <FlatButton
                        label="Cancel"
                        primary={true}
                        onClick={this.handleClose}
                        key="cancel"
                    />
                    <FlatButton
                        type="submit"
                        label="Submit"
                        primary={true}
                        keyboardFocused={true}
                        key="submit"
                    />
                </div>
            </form>
        );
    }
}

const mapDispatchToProps = dispatch => (bindActionCreators({
    addSnippet
}, dispatch));

//Connect the component to the store. See Redux.
export default connect(
    null,
    mapDispatchToProps
)(AddSnippet)
```

My actions file (see redux) :

```js
import {postSnippet} from '../../api/snippets'
import { receiveOneSnippet } from '../'

export function addSnippet(data) {
    return async function (dispatch) {
        try {
            // If  status code == 201 the API return the new created object.
            const response = await postSnippet(data);
            //This object is dispatch to the store in order to update another component
            dispatch(receiveOneSnippet(response))
        } catch (err) {
            //If status code > 299 the payload is catch here.
            throw  err.errors;
        }
    }
}
```

Where my API call is really done (in a separate file) :

```js
export async function postSnippet(data) {
    try {
        const response = await fetch(`mydomain.com/snippets/`, {
            method: 'POST',
            body: JSON.stringify(data),
        });

        //If status code > 299
        if (!response.ok) {
            throw response;
        }
        return await response.json();
    } catch (err) {
        //Throw the return payload
        throw await err.json()
    }
}
```
The result :

![form validation](/img/formik-form-validation.png)
