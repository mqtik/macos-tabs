# MacOS Tabs

## Note

This package is a work in progress. More examples and configuration options will be put up as soon as possible!

## Installation

```bash
npm install --save macos-tabs
```

or

```bash
yarn add macos-tabs
```

## Usage

## API

### MacOSTabs

```javascript
type ID = number | string
type Tabs = Array<Object>

type Props = {
	// onClick event when the user clicks on a tab header
	onAddTabButtonClick?: (e: Event) => void,

	// onClick event when the user click on the close tab button on a tab header
	onCloseTabButtonClick?: (e: Event, closedTabIndex: number) => void,

	// Event when the user stops dragging a header
	// The updated tabs contain the new ordering, you can directly update your state with
	// these returned tabs
	onDragStop?: (e: Event, data: Object, tabs: Tabs, activeTabIndex: number) => void,

	// Event when the user clicks on a tab header
	// The index of the clicked header is passed back
	onTabClick?: (e: Event, index: number) => void,

	// Event when the activeTabIndex is updated internally
	// This allows you to keep track of the activeTabIndex even
	// if you are letting MacOSTabs handle the state
	onSetActiveTab?: (index: number) => void,

	// Specify the position of addTabButton
	addTabPosition: 'none' | 'start' | 'end',

	// Declare whether or not you want to manage the activeTabIndex
	autoActiveTab: boolean,

	// Can be used to set initial active tab index
	// If autoActiveTab is false, then you should have methods
	// to manage the activeTabIndex state
	activeTabIndex: number,

	// Tab content should be wrapped in a TabBody component with the correct props
	// Tabs order can be modified programmatically by manually changing the order
	// of the array
	tabs: Tabs,

	// Declare a div or react component to render when there are no tabs open
	defaultContent: Object,

	// Flag to show or hide tab headers
	showHeader: boolean,

	// Declare the header height in px (note: styles have not been tested with
	// anything but the default of 24px)
	headerHeight: number | string,

	// Declare a custom element that the body should be rendered into
	// instead of directly below the tab headers
	// i.e. <div id="tabBody" />
	customBodyElementId?: string,

	// Experimental/Not Completed
	onMouseEnter?: () => void,
	onMouseLeave?: () => void,
	onDragOut?: (e: Event, data: Object, outTabIndex: number) => void,
	dragOutDistance: number
}

static defaultProps = {
	tabs: [],
	addTabPosition: 'end',
	showHeader: true,
	headerHeight: 24,
	dragOutDistance: 40,
	autoActiveTab: true
}
```

### TabBody

```javascript
type Props = {
	// Label that will be diplayed on the tab header
	label?: string | number,

	// Children components to render
	children?: Object,

	// Unique tabId
	tabId: string | number
}

static defaultProps = {
	label: '',
	children: null
}
```

## Minimal Example

```javascript
// @flow
import React, { Component } from 'react'
import MacOSTabs, { TabBody } from 'macos-tabs'

type Tabs = Array<Object>
type ID = number | string

type Props = Object
type State = {
	tabs: Tabs
}

const defaultStyles = {
	height: '100%'
}

export default class Home extends Component {
	props: Props
	state: State
	id: number

	constructor(props: Props) {
		super(props)

		this.id = 0

		this.state = {
			tabs: [
				this.makeTab(this.id++),
				this.makeTab(this.id++),
				this.makeTab(this.id++)
			]
		}
	}

	makeTab(id: ID) {
		return (
			<TabBody label={ `Test Tab ${ id }` } tabId={ id } key={ id }>
				<div style={{
					height: '100%',
					border: '1px solid red',
					textAlign: 'center',
					boxSizing: 'border-box',
					paddingTop: '20%'
				}}>
					Hello { id }!
				</div>
			</TabBody>
		)
	}

	onDragStop(e: Object, data: Object, tabs: Tabs, activeTabIndex: number) {
		this.setState({
			tabs
		})
	}

	onAddTabButtonClick(e: Object, tabs: Tabs) {
		// Note: Feel free to mutate the returned tabs, it is already deep cloned
		tabs.push(this.makeTab(this.id++))

		this.setState({
			tabs
		})
	}

	onCloseTabButtonClick(e: Object, tabs: Tabs, closedTabIndex: number) {
		// Note: Feel free to mutate the returned tabs, it is already deep cloned
		tabs.splice(closedTabIndex, 1)

		this.setState({
			tabs
		})
	}

	render() {
		return (
			<div style={ defaultStyles }>
				<MacOSTabs
					tabs={ this.state.tabs }
					onDragStop={ this.onDragStop.bind(this) }
					onAddTabButtonClick={ this.onAddTabButtonClick.bind(this) }
					onCloseTabButtonClick={ this.onCloseTabButtonClick.bind(this) }
				/>
			</div>
		)
	}
}
```

## Controlled Active Tab Example

```
// @flow
import React, { Component } from 'react'
import MacOSTabs, { TabBody } from 'macos-tabs'

type Tabs = Array<Object>
type ID = number | string

type Props = Object
type State = {
	tabs: Tabs,
	activeTabIndex: number
}

const defaultStyles = {
	height: '100%'
}

export default class Home extends Component {
	props: Props
	state: State
	id: number

	constructor(props: Props) {
		super(props)

		this.id = 0

		this.state = {
			tabs: [
				this.makeTab(this.id++),
				this.makeTab(this.id++),
				this.makeTab(this.id++)
			],
			activeTabIndex: 0
		}
	}

	makeTab(id: ID) {
		return (
			<TabBody label={ `Test Tab ${ id }` } tabId={ id } key={ id }>
				<div
					style={{
						height: '100%',
						border: '1px solid red',
						textAlign: 'center',
						boxSizing: 'border-box',
						paddingTop: '20%'
					}}
				>
					Hello { id }!
				</div>
			</TabBody>
		)
	}

	onDragStop(e: Object, data: Object, tabs: Tabs, activeTabIndex: number) {
		this.setState({
			tabs,
			activeTabIndex
		})
	}

	onAddTabButtonClick(e: Object, tabs: Tabs) {
		// Note: Feel free to mutate the returned tabs, it is already deep cloned
		tabs.push(this.makeTab(this.id++))

		this.setState({
			tabs,
			activeTabIndex: this.state.tabs.length - 1
		})
	}

	onCloseTabButtonClick(e: Object, tabs: Tabs, closedTabIndex: number) {
		// Note: Feel free to mutate the returned tabs, it is already deep cloned
		tabs.splice(closedTabIndex, 1)

		if (closedTabIndex <= this.state.activeTabIndex) {
			this.setState({
				tabs,
				activeTabIndex: (this.state.activeTabIndex === 0) ? 0 : this.state.activeTabIndex - 1
			})
		} else {
			this.setState({
				tabs
			})
		}
	}

	onSetActiveTab(index: number) {
		this.setState({
			activeTabIndex: index
		})
	}

	render() {
		return (
			<div style={ defaultStyles }>
				<MacOSTabs
					tabs={ this.state.tabs }
					onDragStop={ this.onDragStop.bind(this) }
					onAddTabButtonClick={ this.onAddTabButtonClick.bind(this) }
					onCloseTabButtonClick={ this.onCloseTabButtonClick.bind(this) }
					onSetActiveTab={ this.onSetActiveTab.bind(this) }
					activeTabIndex={ this.state.activeTabIndex }
				/>
			</div>
		)
	}
}
```