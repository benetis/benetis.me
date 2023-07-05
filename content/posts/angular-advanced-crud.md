
---
date : "2017-05-14T23:00:24+02:00"
draft : false
share : true
title : "Let's code: CRUD in Angular with unit tests"
slug : "angular-advanced-crud"
tags : ['Angular', 'Frontend', 'Side-project', "Let's code"]
banner : ""
aliases : ['/angular-advanced-crud/']
menu:
    main:
        parent: 'angular'
---

## Introduction

CRUD application we are going to implement will have some additional complexity you might expect:

- import/export data
- save/load chunks data
- lazy data processing
- pagination
- tests

This project's source code is in github - [https://github.com/benetis/angular-advanced-crud](https://github.com/benetis/angular-advanced-crud)

p.s some of code parts are not in sync going down - so I advise to also fork github repo and check code yourself

## Requirements & Task itself

Aside of the stuff we mentioned above - we obviously will be implementing CRUD operations. Our model on which are going to operate is going to be points (coordinate points `{x, y}`).

- Input points by hand
- Import points from file
- Export points to file
- Clear all points
- Points should be paginated, pagination size can change
- How many squares can be formed from these squares (lazy processing)
- Remove loaded points
- Save chunk of points with specific name (if name exists - overwrite)
- Load chunk of points from saved list
- If something happens (duplicate points ignored...) user needs to be informed
- Delete list of points

We will be using Angular 4.1 with CLI 1.0.1

## Planning UI

Quick mockup on how I imagine this will look like. Probably best starting place for any application which can be componetized.

![](/images/2017/05/points-mock.png)

Clearly we will need a good grid/table component. I have implemented tables so many times in Angular that this time I am just gonna skip it. There is even a post on Angular table by me - [[https://benetis.me/posts/angular-table/](/posts/angular-table/)

Tabs component to switch between points and squares. (Since squares need pagination) OR just put squares table below.

Action menu for actions to perform on selected rows of table

Validations for inputs to add to the table. We have limits

Research into square finding algorithm - but I guess we can leave that for later.

Import file and upload to table can be plugged anywhere

After retro - I have updated mockup with another possible solution:

![](/images/2017/05/updated-mockup.png)

It doesn't really matter for us - we can decide this later.

## Architecture of code

We have a mockup, requirements. Now we can look at what components, models we will need.

I like to start from data since everything revolves around it. Starting from the bottom.

### Main types

Point and Square.

```typescript
interface Point {x: number, y: number}
// Corner 1, Corner 2
interface Square {q1: Point, q2: Point, q3: Point, q4: Point}

// Self explanatory, just for planning
type ListOfPoints = { name: string, points: Point[] }
type ListOfSquares = Square[]
```

### Modules, components, services

We can live with one module - app module. No reason to complicate ourselves when task is simple

**Services:**

- import/export data
- IO data transformer
- square-finder
- favorite-points

**Components:**

- table
    - pagination
- action-menu
- delete button
- save-list-button
- replace-list-button
- inputs
- points-table
- squares-table

We might have forgotten something - but for now it seems like everything. Stuff can be added later.

### Notorious table

Certainly table is one of the most popular components (everywhere?). I had a "pleasure" to create two tables (different API) in Angular and this time we will use [http://swimlane.github.io/ngx-datatable](http://swimlane.github.io/ngx-datatable). We will not wrap it inside of our component just for the sake simplicity, however I certainly recommend for you to do so.

To install:
`npm i @swimlane/ngx-datatable --save`

Creating skeleton for points-table using Angular CLI. Everything under `/src/app`

`ng g component points-table`

Compile application. Run tests.

We will need some dummy data, columns. We should think in terms of observables since we will be fetching data from this component directly.

points-table.component.ts
```typescript
public points$: Observable<Point[]> = of([
    {x: 1, y: 2},
    {x: -12, y: 22},
    {x: -2222, y: 4999},
]);
```
points-table.component.html
```html
<h2>Points table</h2>
<ngx-datatable
  [rows]="points$ | async"
  [columns]="[{name:'X'},{name:'Y'}]"
  [limit]="10"
>
</ngx-datatable>
```
And we can already see something:

![](/images/2017/05/table-initial.png)


### Adding a points

Since we will be adding a point - we need a service to handle this for us. We want to subscribe to that service for points to be updated.

Few tests TDD style and we should have basic service for getting points data.

```typescript

@Injectable()
export class PointService {

    private points = new BehaviorSubject([])
    private _points = []

    constructor() {
        this.points.subscribe(_ => this._points = _)
    }

    public getPoints(): Observable<Point[]> {
        return this.points;
    }

    public addPoints(pointsToAdd): Observable<Point[]> {
        this.points.next([...this._points, ...pointsToAdd])
        return this.getPoints()
    }

}

```

and their tests:

```typescript
it('should return empty observable when called getPoints', done => {
    service.getPoints().subscribe(p => {
        expect(p).toEqual([]);
        done();
    })
})

it('should add points and return all points', done => {
    const pointsToAdd = [
        {x: 1, y: 1},
        {x: 0, y: 1},
    ]

    service.addPoints(pointsToAdd).subscribe(p => {
        expect(p).toEqual(pointsToAdd);
        done();
    })
})

it('should add points twice and return all points', done => {
    const pointsToAdd = [
        {x: 1, y: 1},
        {x: 0, y: 1},
    ]

    service.addPoints(pointsToAdd)
    service.addPoints(pointsToAdd)

    service.getPoints().subscribe(p => {
        expect(p).toEqual([...pointsToAdd, ...pointsToAdd]);
        done();
    })
})
```

You can find these files in github - [https://github.com/benetis/angular-advanced-crud/blob/master/src/app/points-service.service.ts](https://github.com/benetis/angular-advanced-crud/blob/master/src/app/points-service.service.ts)

Of course instead of grabbing points inside points-table we will now need to subscribe for them from service.

Next - AddPoint component. Put inputs inside that component and after clicking the button - just add points to our points service.

Important: component like `addPoint` if possible should be made dumb. You always want to have as many dumb components as possible. It means it shouldnt inject pointsService, but instead let parent component handle that for us. This will enable easier testing

This gives us a little bit of separation since we will need to handle validation here as well.

```html
<form>
    <label>X
        <input type="text" ngModel #x name="x">
    </label>
    <label>Y
        <input type="text" ngModel #y name="y">
    </label>
    <button (click)="addPoint(x.value, y.value)">Add</button>
</form>
```

```typescript
    public addPoint() {
        this.addPoints.emit([{x: this.x, y: this.y}])
    }
```

And of course validations. Limits are:

- integers only
- min -5000, max +5000

Being lazy - we can just install this library to provide custom validators:

`npm install ng2-validation --save`

```html
<form #addPointForm="ngForm">
    <label>X
        <input type="text"
               [(ngModel)]="x"
               #field="ngModel"
               name="x"
               required
               number
               [min]="-5000"
               [max]="5000">
    </label>
    <label>Y
        <input type="text"
               [(ngModel)]="y"
               name="y"
               required
               number
               [min]="-5000"
               [max]="5000">
    </label>
    <button (click)="addPoint(x, y)">Add</button>
</form>
<p *ngIf="!addPointForm.form.valid && addPointForm.form.dirty">
    X and Y need to be between -5000 and 5000
</p>
```

![](/images/2017/05/validation.gif)

Next step is to validate if point to add doesn't already exist in list. (No duplicates) + list cannot get bigger than 10000.

```typescript
export interface PSResponse {
    message: string,
    error: boolean,
    point?: Point
}
```

New type will hold response from `addPoint` to handle two conditions we defined above.

Begin by updating tests in points-service. Check them here - [https://github.com/benetis/angular-advanced-crud/blob/master/src/app/points-service.service.spec.ts](https://github.com/benetis/angular-advanced-crud/blob/master/src/app/points-service.service.spec.ts)

As for code - we ended up with this:

```typescript
public addPoints(pointsToAdd: Point[]): Observable<PSResponse[]> {
        const limit = 10000
        const currentSize = this._points.length
        const union = _.differenceWith(pointsToAdd, this._points, this.isEqual)
        const xor = _.isEqual(union, this._points) ? [] : union

        const duplicates = _.intersectionWith(this._points, pointsToAdd, this.isEqual)
            .map(p => ({
                error: true,
                message: 'duplicate',
                point: p
            }))

        const toAddSize = pointsToAdd.length
        const overLimit: boolean = (currentSize + toAddSize) > limit
        if (overLimit) {
            const canBeImported: number = Math.abs(limit - currentSize - toAddSize)
            this.points.next([...this._points, ...union.slice(0, canBeImported)])

            return of([
                ...duplicates,
                {error: true, message: 'over limit'}
            ])
        } else {
            this.points.next(this._points.concat(xor))
            return of([
                ...duplicates,
            ])
        }

    }
```

Let's hope to points importing and come back to table a little bit later

## Points importing

Check [https://caniuse.com/#search=file](https://caniuse.com/#search=file) to see if File API is supported. Which is good enough

Create `import-gatekeeper` component to hold our input where file will be uploaded. If file is changed - will output its contents out :)

```typescript
export class ImportGatekeeperComponent implements OnInit {

    @Output()
    public fileContents: EventEmitter<any[]> = new EventEmitter();

    constructor() {
    }

    ngOnInit() {
    }

    public fileChange($event) {
        this.readFile($event.target)
    }

    private readFile(inputValue) {
        const file: File = inputValue.files[0];
        const myReader: FileReader = new FileReader();

        myReader.onloadend = e => {
            this.fileContents.emit(
                myReader
                    .result
                    .split('\n')
                    .filter(_ => _ !== '')
                    .map(line => {
                        const [x, y] = line.split(' ')
                        return {x: +x, y: +y}
                    })
                )
        }

        myReader.readAsText(file);
    }

}
```

In our wrapper (app.component) connect callback function to addPoints
```typescript
public fileUploaded(contents: any[]) {
    this.importResponses$ = this.pointService.addPoints(contents)
}
```

If file is uploaded - data is imported! All good - we get a response with what happened also! We can pass it to import gatekeeper for it to handle.

![](/images/2017/05/import-response.gif)

### Cosmetics

App is shaping up. Time to add styles, check what is redundant.


![](/images/2017/05/presentation.png)

Little updates to UI just to have everything a little bit more organized. We can come back to it later if there is need, but this is not our focus.

### Action menu and multi select

We want to select multiple table rows and perform actions on them:

- Save them as another list
- Delete them

For delete we need to update our points service to support deleting.

Important moment - we should add id field to our data model to have something "easier" for identification of our object. However, since it is a simple point - we will skip this step.

In points-table add two functions
```typescript
    public delete(e) {
        this.pointsService.deletePoints(this.selected)
        this.selected = []
    }

    public onSelect({selected}) {
        this.selected.splice(0, this.selected.length);
        this.selected.push(...selected);
    }
```

and in our data handler just xor

```typescript
  public deletePoints(pointsToDelete: Point[]): Observable<boolean> {
        this.points.next(_.xorWith(pointsToDelete, this._points, this.isEqual))

        return of(true);
    }
```


![](/images/2017/05/delete.gif)

### Storing chunks of data

Saving list of points and giving that list name a name. If name exists - overwrite.

Creating a new component to handle all operations related to lists. Selected rows will be outputted from points table and passed into new component

Next is simple:

```typescript
export class FavoritePointsComponent implements OnInit {

    @Input() selected: Point[] = []

    public savedListsOfPoints: { [key: string]: Point[] } = {}

    constructor() {
    }

    ngOnInit() {
    }

    public saveList(listName) {
        this.savedListsOfPoints[listName] = this.selected
    }

    public keys(obj) {
        return Object.keys(obj)
    }

}
```

```html
<div class="list-menu">
    <div class="box-sm">
        <p>Save selected points to list</p>
        <input type="text"
               #listName/>
        <button (click)="saveList(listName.value)">Save list</button>
    </div>
    <div class="box-sm">
        <p>Load list points to table</p>
        <select>
            <option
                    *ngFor="let pointsList of keys(savedListsOfPoints)"
                    [value]="pointsList">{{pointsList}}
            </option>
        </select>
        <button (click)="loadList($event)">Load list</button>
        <button (click)="deleteList($event)">Delete list</button>
    </div>
</div>
```

We use object to store our lists since we will overwrite lists (by name). Else is self explanatory

![](/images/2017/05/save-list.gif)

### Loading saved lists into table

Table list will be replaced by user chosen saved list.

We need `setPoints` method.

```typescript
    public setPoints(points: Point[]): Observable<boolean> {
        this.points.next(points)
        return of(true);
    }
```

![](/images/2017/05/load-list.gif)

## Finding squares

Creating separate table to hold our squares.

Grabbing algorithm from here [http://www.geeksforgeeks.org/check-given-four-points-form-square/](http://www.geeksforgeeks.org/check-given-four-points-form-square/)

Seems to work good enough for this exercise.

We don't care that much about performance - our limits are n < 10000 which can be processed quickly. We care that it will be displayed lazy so user can see as it processes

Squares will be from list of points. That being said we can create a new service which will inject `PointService` to get current list squares and start counting

Code is here - [https://github.com/benetis/angular-advanced-crud/blob/master/src/app/squares.service.ts](https://github.com/benetis/angular-advanced-crud/blob/master/src/app/squares.service.ts)

![](/images/2017/05/find-squares.gif)

## Summary

The thing you want you will find in code, here is just a quick write up on this task :)
