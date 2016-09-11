# Dev Environment Configuration

# Use Admin Component
## Main App
```js
import 'babel-polyfill';
import React from 'react';
import { render } from 'react-dom';

import FakeRest from 'fakerest';
import fetchMock from 'fetch-mock';
import data from './data';

import { simpleRestClient, Admin, Resource } from 'admin-on-rest';
import { Delete } from 'admin-on-rest/mui';

import { DeviceList, DeviceEdit, DeviceCreate, DeviceIcon } from './devices';
import { UserList, UserEdit, UserCreate, UserIcon } from './users';
import { RoleList, RoleEdit, RoleCreate, RoleIcon } from './roles';
import { ItemList, ItemEdit, ItemCreate, ItemIcon } from './items';

const restServer = new FakeRest.FetchServer('http://localhost:3000');
restServer.init(data);
restServer.toggleLogging(); // logging is off by default, enable it
fetchMock.mock('^http://localhost:3000', restServer.getHandler());

const restClient = simpleRestClient('http://localhost:3000');
const delayedRestClient = (type, resource, params) => new Promise(resolve => setTimeout(() => resolve(restClient(type, resource, params)), 500));

render(
    <Admin restClient={delayedRestClient} title="IOT Data">
        <Resource name="users" list={UserList} edit={UserEdit} create={UserCreate} remove={Delete} icon={UserIcon} />
        <Resource name="roles" list={RoleList} edit={RoleEdit} create={RoleCreate} remove={Delete} icon={RoleIcon} />
        <Resource name="devices" list={DeviceList} edit={DeviceEdit} create={DeviceCreate} remove={Delete} icon={DeviceIcon} />
        <Resource name="items" list={ItemList} edit={ItemEdit} create={ItemCreate} remove={Delete} icon={ItemIcon} />
    </Admin>,
    document.getElementById('root')
);
	
```
## Define Entity Component (use user as example)
```js
import React from 'react';
import { List, Filter, Edit, Create, Datagrid, DateField, TextField, EditButton, DisabledInput, TextInput, LongTextInput, EmailInput, PasswordInput, DateInput, ReferenceManyField, ReferenceField, ReferenceInput,SelectInput} from 'admin-on-rest/mui';

export UserIcon from 'material-ui/svg-icons/social/person';

const UserFilter = (props) => (
    <Filter {...props}>
        <TextInput label="Search" source="q" alwaysOn />
        <TextInput label="Username" source="username" />
    </Filter>
);

export const UserList = (props) => (
    <List {...props} filter={UserFilter}>
        <Datagrid>
        <TextField label="id" source="id" />
            <ReferenceField label="role" source="role_id" reference="roles">
                <TextField source="title" />
            </ReferenceField>
	
            <TextField label="username" source="username" />
            <TextField label="fullname" source="fullname" />
            <TextField label="email" source="email" />
            <EditButton />
        </Datagrid>
    </List>
);

//        <ReferenceManyField label="roles" reference="userroles" target="user_id">
//                <TextField source="role" />
//        </ReferenceManyField>


const UserUsername = ({ record }) => {
    return <span>User {record ? `"${record.username}"` : ''}</span>;
};

export const UserEdit = (props) => (
    <Edit username={UserUsername} {...props}>
        <DisabledInput label="Id" source="id" />
        <TextInput label="Username" source="username" />
        <TextInput label="Full Name" source="fullname" options={{ multiLine: true }} />
        <LongTextInput label="Email" source="email" />
	<ReferenceInput label="Role" source="role_id" reference="roles">
            <SelectInput optionText="title" />
        </ReferenceInput>
        <PasswordInput label="Password" source="password" />
    </Edit>
);

export const UserCreate = (props) => (
    <Create {...props}>
        <TextInput label="Username" source="username" />
        <TextInput label="Full Name" source="fullname" options={{ multiLine: true }} />
        <LongTextInput label="Email" source="email" />
        <ReferenceInput label="Role" source="role_id" reference="roles" allowEmpty>
            <SelectInput optionText="title" />
        </ReferenceInput>    
    </Create>
);
```

# Add Component to Template

## Example 1 (add PasswordField to the template)
* Create your react Component
```js
import React, { Component, PropTypes } from 'react';
import TextField from 'material-ui/TextField';

const EMPTY_STRING = '';

class PasswordInput extends Component {
    handleChange(event) {
        this.props.onChange(this.props.source, event.target.value);
    }

    render() {
        const { source, label, record, options } = this.props;
        return (<TextField
	    type="password"
            name={source}
            floatingLabelText={label}
            value={record[source] || EMPTY_STRING}
            onChange={::this.handleChange}
            {...options}
        />);
    }
}

PasswordInput.propTypes = {
    source: PropTypes.string.isRequired,
    label: PropTypes.string,
    record: PropTypes.object,
    options: PropTypes.object,
    onChange: PropTypes.func,
    includesLabel: PropTypes.bool.isRequired,
};

PasswordInput.defaultProps = {
    record: {},
    options: {},
    includesLabel: true,
};

export default PasswordInput;
```
* register your component in index.js
```js
// index.js
export PasswordInput from './PasswordInput';
```
