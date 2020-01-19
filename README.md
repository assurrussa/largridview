# Package LarGridView

A simple package for the view of the grid on Vue. It is easily customizable and easy to work. The main task of the grid package takes over. You do the rest.

## Installation

`npm install largridview`

## Usage

```
  import Grid from 'largridview';

  <grid :url="'api/all'"
        :fields="[{width: '10%', name: 'id', label: '', isSort: true}, {width: null, name: 'action', label: 'Действия', isAction: true}]"
        @response="onResponse"
        :isBusy="loadingSendingStatus"
        sort-key='id'>
        <tr v-for="(item, index) in items" :key="index">
            ...
        </tr>
  </grid>
```