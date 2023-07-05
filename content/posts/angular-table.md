
---
date : "2016-11-06"
draft : false
title : "Let's code: Creating functional table with Angular"
slug : "angular-table"
tags : ['Angular2', 'Frontend', 'Side-project', "Let's code"]
banner : ""
aliases : ['/angular-table/']
menu:
    main:
        parent: 'angular'
---

##### Offtopic
Currently volunteering in organisation called "Erasmus student Network Lithuania". Working with a team to develop internal HR system.

##### Intro

Looking at the requirement - there is a need to display data. A list of users. And some actions which can be done with each of the user. Table is a good way to make data look good.

Let's try to design how could this table look.

##### Basic requirements

* Table might need to display hundreds of records - pagination
* No need for selecting rows
* There will be action buttons in each row to interact with user
* Inline editing
* Should look nice (we don't really care about design, just nice is enough)
* Filtering
* Sorting

##### Attempt #1

Using bootstrap - just the markup with some `*ngFor` magic.

![](/images/2017/02/Screen_Shot_2016_11_06_at_17_06_50.png)

```html
<table class="table table-striped table-hover">
  <thead>
  <tr>
    <th>First Name</th>
    <th>Surname</th>
  </tr>
  </thead>
  <tbody>
  <tr *ngFor="let user of users"
  >
    <td>{{user.name}}</td>
    <td>{{user.surname}}</td>
    <td>{{user.section}}</td>
  </tr>
  </tbody>
</table>
```

What is cool about this - just because of `table-striped` class we get odd/even rows and because of `table-hover` class rows have hover effect. While this is simple enough - we don't have pagination, filtering and sorting - and this would require us put some real angular code. While using helpers from `ng-bootstrap` like pagination could speed us up - but there should be a more complete solution just for the table.

##### Attempt #2

There are other more complete solutions to the table like: `angular2-data-table` or `ng2-table` - which has all the features we discussed earlier.

`angular2-data-table` looks like more successful project (and has more features), but I have already tried it out - so let's go with `ng2-table`.
_If you are choosing between those two - go with `angular2-data-table`_.

There is enough documentation of table to just copy and adjust the code to have a working example. It is just that in example it uses pagination of `ng2-bootstrap`, while I am using `ng-bootstrap`. For consistency sake I changed few lines and it works like a charm. Although the table itself could be transformed to component - we can skip this until we need another table.
You can check code:

* [manage-users.component.html](https://github.com/ESNLithuania/boarded/blob/401cd4ca9595417ea0eea58279476a370d7cb7ba/src/app/modules/dashboard/manage-users.component.html)
* [manage-users.component.ts](https://github.com/ESNLithuania/boarded/blob/401cd4ca9595417ea0eea58279476a370d7cb7ba/src/app/modules/dashboard/manage-users.component.ts)

Table currently looks like this -

![](/images/2017/02/Screen_Shot_2016_11_06_at_20_16_20.png)

Let's check what requirements we fulfilled -

* [x] Table might need to display hundreds of records - pagination
* [x] No need for selecting rows
* [ ] There will be action buttons in each row to interact with user
* [ ] Each row will have inline editing
* [x] Should look nice (we don't really care about design, just nice is enough)
* [x] Filtering
* [x] Sorting

And there seems to be no good way to have inline-editing or buttons. After checking project code - it seems to be really old (using `[innerHtml]`), without tests - probably abandoned.

##### Attempt #3

Let's get rid of ng2-table. Use it's API ideas and just implement simple functionality we need. No need to make it reusable.

###### Table template

Let's bring back the good ol' bootstrap in attempt #1 one and few enhancements.

First we want to have columns as an for easier sorting and etc. While we are at it - we can introduce interface Column just for better types.
Same idea with rows instead of users. Easier to manipulate later for us.

```html
<table class="table table-striped table-hover">
  <thead>
  <tr>
    <th *ngFor="let column of columns">{{column.title}}</th>
  </tr>
  </thead>
  <tbody>
  <tr *ngFor="let row of rows"
  >
    <td *ngFor="let column of columns">
      {{getData(row, column)}}
    </td>
  </tbody>
</table>
```

Pagination works, table loads.

###### Filtering

We are now not depending on ng2-table anyyway, except its' API and documentation.

We want our all fields filter to work to filter our rows.

```html
<input *ngIf="config.filtering"
           placeholder="Filter all columns"
           [(ngModel)]="config.filtering.filterString"
           class="form-control"
           (keydown)="onChangeTable(config)"/>
```

Changed the directive of ng2 table to `ngModel` - to bind it to filtering string + custom change event - changed it to keypress and we have desired functionality. So far so good.

![](/images/2017/02/giphy--3-.gif)

##### Sorting

From the api it seems that columns need to have `sort` property which is either `'desc', 'asc', ''(unsorted)`

Let's make it a little more cleaner by limiting string in our interface:
```typescript
interface Column {
  title: string,
  name: string,
  sort? : 'desc' | 'asc' | ''
}
```

Sorting function seems complicated -

```typescript
  public changeSort(data: any, config: any): any {
    if (!config.sorting) {
      return data;
    }
    let columns = this.config.sorting.columns || [];

    let columnName: string = void 0;
    let sort: string = void 0;

    for (let i = 0; i < columns.length; i++) {
      if (columns[i].sort !== '' && columns[i].sort !== false) {
        columnName = columns[i].name;
        sort = columns[i].sort;
      }
    }

    if (!columnName) {
      return data;
    }

    // simple sorting
    return data.sort((previous: any, current: any) => {
      if (previous[columnName] > current[columnName]) {
        return sort === 'desc'
          ? -1
          : 1;
      } else if (previous[columnName] < current[columnName]) {
        return sort === 'asc'
          ? -1
          : 1;
      }
      return 0;
    });
  }
```

Let's see how can we make this more simple by keeping the functionality

```typescript
 public changeSort(data: any, config: any): any {
    if (!config.sorting) {
      return data;
    }
    const columns = this.config.sorting.columns || [];

    const columnWithSort: Column = columns.find((column: Column) => {
      /* Checking if sort prop exists and column needs to be sorted */
      if(column.hasOwnProperty('sort') && column.sort !== '') {
        return true;
      }
    });

    return data.sort((previous: any, current: any) => {
      if (previous[columnWithSort.name] > current[columnWithSort.name]) {
        return columnWithSort.sort === 'desc'
          ? -1
          : 1;
      } else if (previous[columnWithSort.name] < current[columnWithSort.name]) {
        return columnWithSort.sort === 'asc'
          ? -1
          : 1;
      }
      return 0;
    });
  }
```

Sorting seems to be working - just that there are no events which trigger sorting. Now only works from initial config.

What is needed - upon clicking on <th> it should sort that column showing icons needed.

```html
    <th *ngFor="let column of columns" (click)="sortByColumn(column)">
```

Triggers the function the new function I just wrote - which updated the columns sorting

```typescript
  public sortByColumn(columnToSort: Column) {
    const sorting: Array<Column> = Object.assign({}, this.config.sorting).columns;

    const sorted = sorting.map((column: Column) => {
      if (columnToSort.name === column.name) {
        const newSort = column.sort === 'asc'
          ? 'desc'
          : 'asc';
        return Object.assign(column, {sort: newSort});
      } else {
        return Object.assign(column, {sort: ''});
      }
    });

    const config = Object.assign({}, this.config, {
      sorting: {columns: sorted}
    });
    this.onChangeTable(config);
  }
```

All is left to add some icons to indicate currently sorted columns. Bootstrap 4 currently has this nice "TODO" in Icon fonts. Let's go with font-awesome.

```html

      <i *ngIf="columnSortWay(column) !== ''" class="fa fa-sort-alpha-{{columnSortWay(column)}}" aria-hidden="true">
      </i>
```

Looks good! And changes on click.
![](/images/2017/02/Screen_Shot_2016_11_06_at_23_07_51.png)

##### Continuing

Let's look at our requirements list.

* [x] Table might need to display hundreds of records - pagination
* [x] No need for selecting rows
* [ ] There will be action buttons in each row to interact with user
* [ ] Inline editing
* [x] Should look nice (we don't really care about design, just nice is enough)
* [x] Filtering
* [x] Sorting

We are at same position as before - its just that we have as much freedom as we want to upgrade our table!

##### Inline editing

Next we want inline editing. Users clicks edit icon - all fields become editable. Once user click out - they become static again and field is saved.

Let's do it the simple way.

Icon itself

```html
      <i class="fa fa-pencil-square row-icon"
         (click)="editRow(i)"
      >
```

If to display input for editable row. For now let's go with simple variable - we just need to keep number of row we are editing.

```html
<div *ngIf="editableRowNumber === i">
        <input [ngModel]="getData(row, column)">
      </div>
      <div *ngIf="editableRow !== i">
        {{getData(row, column)}}
</div>
```

And for this I am going to hell. Simple solution for editing cancellation?
```typescript
  @HostListener('document:click', ['$event.target'])
  public onClick(targetElement: HTMLElement) {
    const tagsNotToInteractWith = ['INPUT', 'I', 'TH'];
    const interact = tagsNotToInteractWith.reduce((result, item) => {
      return result && targetElement.tagName != item;
    }, true);

    if(interact) {
      this.updateUser();
      this.editableRowNumber = -1;
    }

  }
```

Yep. Catching all clicks and checking if they are not th (sorting), input (all inputs we want to ignore) or i (icons, edit-icon). Basic version should work for now. _NOTE_ - we probably can do something smarter with ViewChild checking if it inside/outside of component, but we want to cancel in component also.

Will make a good post in the future to refactor!


![](/images/2017/02/giphy--4-.gif)

We have the UI working for inline editing - what is left is saving the data.

Let's do it a fast way (will definitely need refactor - but that is what love) -
```typescript
  private editableRow: Array<any> = [];
```

```typescript
  public updateEditableRowData(data: any, column: Column) {
    this.editableRow[column.name] = data;
  }
```

and

```html
        <input #input [value]="getData(row, column)" (blur)="updateEditableRowData(input.value, column)">

```

Now UI is outputting the values we need and we have row number in component state.

Lastly, let's just merge old and new user data:
```typescript
  private updateUser() {
    if(this.editableRowNumber !== -1) {
      const originalUser = this.rows[this.editableRowNumber];
      const updatedUser = <User>Object.assign(originalUser, this.editableRow);
       this.userService
        .updateUser(updatedUser)
    }
  }
```

![](/images/2017/02/giphy--5-.gif)

The way service is implemented I'll leave for another post. However if you are interested you can check code here -
[request.service.ts](https://github.com/ESNLithuania/boarded/blob/3f33924df738580cece1c32f9c4e13e50c732ec3/src/app/services/request.service.ts)
[user.service.ts](https://github.com/ESNLithuania/boarded/blob/3f33924df738580cece1c32f9c4e13e50c732ec3/src/app/services/user.service.ts)

If we check our requirements list

* [x] Table might need to display hundreds of records - pagination
* [x] No need for selecting rows
* [x] There will be action buttons in each row to interact with user
* [x] Inline editing
* [x] Should look nice (we don't really care about design, just nice is enough)
* [x] Filtering
* [x] Sorting

It seems we have filled all of them. All is left are some small tweaks.

Until next time!

### Feedback

If you have any suggestions - I am eagerly waiting for feedback. [https://benetis.me/posts/contact-me/](/posts/contact-me/)
