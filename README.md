# Package LarGridView

### LarGridView package for working with grid view in Laravel with Vue.js

A simple package to view the grid on Vue with the Laravel web framework. Easy to configure and easy to operate. The main task of the grid package takes on. You do the rest.

## Installation

`npm install largridview`

## Usage

```
  import Vue from 'vue'
  
  import * as LarGridView from 'largridview'
  Vue.use(LarGridView)

...

  <LarGridView :url="'api/all'"
        :fields="[{width: '10%', name: 'id', label: '', isSort: true}, {width: null, name: 'action', label: 'Actions', isAction: true}]"
        @response="onResponse"
        :isBusy="loadingSendingStatus"
        sort-key='id'>
        <tr v-for="(item, index) in items" :key="index">
            ...
        </tr>
  </LarGridView>
```


## Props

### Stack

| Name | Type | Required | Default | Description |
| --- | --- | --- | --- | --- |
| `url` | String | `Yes` | `'/'` | URL to get the required data. If the line is empty, then data will not be sent. |
| `fields` | Array | `Yes` | `[]` | Fields to display. Example: `[{width: '10%', name: 'id', label: '', isSort: true}, {width: null, name: 'action', label: 'Actions', isAction: true}]`. |
| `dataPath` | String | No | `data.list` | The path to receiving data from a pagination request response. |
| `data` | Object | No | `{}` | It is necessary in cases when you need to control some data for sending. |
| `isSend` | Boolean | No | `true` | Enable automatic data sending. |
| `isBusy` | Boolean | No | `false` | Loading data into a table. |
| `isLoadingOn` | Boolean | No | `true` | Enable basic data loading. |
| `isSearch` | Boolean | No | `false` | Enable table search. |
| `isPerPageOn` | Boolean | No | `true` | Enable selection of the number of table rows per page. |
| `isPaginateOn` | Boolean | No | `true` | Enable pagination. |
| `isPaginateSimple` | Boolean | No | `false` | Simple pagination. |
| `isPaginateAuto` | Boolean | No | `true` | Enable automatic pagination detection - simple pagination or not. |
| `sortKey` | String | No | `null` | Default Sort Column. |
| `perPage` | Array | No | `[20, 50, 100, 200]` | The number of rows for the table. |
| `perPageDefault` | Number | No | `20` | Default row count for table. |
| `idTable` | String | No | `tableID` | Table id for better control. |
| `classTable` | String | No | `` | Class for the table itself. |
| `classBlock` | String | No | `` | Class for the block itself. |
| `placeholderFilterSearch` | String | No | `Search...` | Placeholder for search input. |
| `textFromTo` | String | No | `Showing :from to :to` |  |
| `textTotal` | String | No | `of :total` |  |
| `textNext` | String | No | `Next ›` |  |
| `textPrev` | String | No | `‹ Prev` |  |
| `paginateDeltaLeft` | Number | No | `The number of pages to the left of the current page` |  |
| `paginateDeltaRight` | Number | No | `The number of pages to the right of the current page` |  |


## Events

| Name | Type | Description |
| --- | --- | --- |
| `loading` | Boolean | Data load status change event. |
| `response` | Object | Event when receiving data from the server. |
| `filter:select` | Number | The event of selecting the number of lines displayed on the page. |
| `filter:sort` | Object | Column sort selection event. |
| `filter:search` | String | The event of entering text into the search string. |
| `page:select` | String | The event of clicking on the pagination button on. |
| `page:next` | String | Pagination forward button click event. |
| `page:prev` | String | Pagination back button click event. |
