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
        :fields="[{width: '10%', name: 'id', label: '', isSort: true}, {width: null, name: 'action', label: 'Действия', isAction: true}]"
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
| `url` | String | Yes | `'/'` | URL to get the required data. |
| `fields` | Array | Yes | `[]` | Fields to display. Example: `[{width: '10%', name: 'id', label: '', isSort: true}, {width: null, name: 'action', label: 'Действия', isAction: true}]`. |
| `dataPath` | String | No | `data.list` | The path to receiving data from a pagination request response. |
| `isBusy` | Boolean | No | `false` | Loading data into a table. |
| `isSearch` | Boolean | No | `false` | Enable table search. |
| `isPaginateOn` | Boolean | No | `true` | Enable pagination. |
| `isPaginateSimple` | Boolean | No | `false` | Simple pagination. |
| `isPaginateAuto` | Boolean | No | `true` | Enable automatic pagination detection - simple pagination or not. |
| `sortKey` | String | No | `null` | Default Sort Column. |
| `perPage` | Array | No | `[20, 50, 100, 200]` | The number of rows for the table. |
| `perPageDefault` | Number | No | `20` | Default row count for table. |
| `idTable` | String | No | `tableID` | Table id for better control. |
| `classTable` | String | No | `jambo_table bulk_action table-striped table-bordered` | Class for the table itself. |
| `classBlock` | String | No | `dataTables_wrapper form-inline dt-bootstrap` | Class for the block itself. |
| `placeholderFilterSearch` | String | No | `Search...` | Placeholder for search input. |
| `textFromTo` | String | No | `Showing :from to :to` |  |
| `textTotal` | String | No | `of :total` |  |
| `textNext` | String | No | `Next ›` |  |
| `textPrev` | String | No | `‹ Prev` |  |