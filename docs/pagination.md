# Pagination in NGXS Firestore

> With query cursors in Cloud Firestore, you can split data returned by a query into batches according to the parameters
> you define in your query.

## Using `NgxsFirestorePagination`

The service connect method takes as arguments the Action that will trigger subscribing to the Firestore query, it added
new items to state array. In addition an opts object with to field, to pass the function that returns the Firestore
query.

```ts
import { NgxsFirestorePagination } from '@ngxs-labs/firestore-plugin';
import { Injectable } from '@angular/core';
//...

@Injectable({
    providedIn: 'root'
})
export class RacesFirestorePagination extends NgxsFirestorePagination<Race> {
    limit = 5;
    path = 'races';
    orderBy = 'created_at'; //document firestore interface
    orderByDirection = 'desc'; //asc or desc
    format = (data) => data; //Each document format.
}
```

```ts
//...
import { NgxsFirestoreConnect } from '@ngxs-labs/firestore-plugin';
import { NgxsFirestorePagination } from '@ngxs-labs/firestore-plugin';
import { append, patch } from '@ngxs/store/operators';

import { PayloadFetch } from '@ngxs-labs/firestore-plugin';

export class Get {
    public static readonly type = '[Races] Get By Page';
}

export class Fetch {
    public static readonly type = '[Races] Fetch';
    constructor(public payload?: PayloadFetch) {}
}

export class PaginationRacesState implements NgxsOnInit {
    @Selector() static races(state: RacesStateModel) {
        return state.races;
    }
    constructor(private racesFS: RacesFirestorePagination, private ngxsFirestoreConnect: NgxsFirestoreConnect) {}

    ngxsOnInit() {
        this.ngxsFirestoreConnect.connect(RacesByPageActions.Get, {
            to: () => this.racesFS.collection$()
        });
    }

    @Action(StreamConnected(RacesByPageActions.Get))
    getConnected(ctx: StateContext<RacesStateModel>, { action }: Connected<RacesByPageActions.Get>) {
        // do something when connected
    }

    @Action(StreamEmitted(RacesByPageActions.Get))
    getEmitted(ctx: StateContext<RacesStateModel>, { action, payload }: Emitted<RacesByPageActions.Get, Race[]>) {
        ctx.setState(
            patch({
                races: append([...payload])
            })
        );
    }

    @Action(StreamDisconnected(RacesByPageActions.Get))
    getDisconnected(ctx: StateContext<RacesStateModel>, { action }: Disconnected<RacesActions.Get>) {
        // do something when disconnected
    }

    @Action(RacesByPageActions.Fetch)
    fetch(ctx: StateContext<RacesStateModel>, { payload }: RacesByPageActions.Fetch) {
        this.racesFS.fetch(payload);
    }
}
```