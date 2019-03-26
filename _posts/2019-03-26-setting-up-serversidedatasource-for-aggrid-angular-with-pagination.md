---
layout: post
hidden: false
title: Setting Up ServerSideDatasource for agGrid Angular with Pagination
tags: []
---
😫, agGrid. If you've been facing issues properly setting up agGrid Angular for server-side data fetching, filtering, etc, then you're not alone.

After a few tries I finally got it working, the trigger being these function from their [Infinite Scroll example](https://www.ag-grid.com/javascript-grid-server-side-model-infinite/) which shows how to pick up request parameters passed from the grid.

```
function ServerSideDatasource(server) {
  return {
    getRows(params) {
      setTimeout(function() {
        var response = server.getResponse(params.request);
        if (response.success) {
          params.successCallback(response.rows, response.lastRow);
        } else {
          params.failCallback();
        }
      }, 500);
    }
  };
}

function FakeServer(allData) {
  return {
    getResponse(request) {
      console.log("asking for rows: " + request.startRow + " to " + request.endRow);
      var rowsThisPage = allData.slice(request.startRow, request.endRow);
      var lastRow = allData.length <= request.endRow ? data.length : -1;
      return {
        success: true,
        rows: rowsThisPage,
        lastRow: lastRow
      };
    }
  };
}
```

So here's what I did:

```
public onReady($event) {
        this.gridOptions = {
            ...this.gridOptions,
            ...$event,
        };
        this.calculateRowCount();

        // this.fetchData(this.gridOptions);

        const _that = this;
        const dataSource: IServerSideDatasource = {
            getRows: function(params) {
                console.log('request params ', params);
                _that.previousRequestParams = params;
                console.log('asking for ' + params.request.startRow + ' to ' + params.request.endRow);
                setTimeout(function() {

                    _that.contactsService.getAll({
                        perPage: params.request.endRow - params.request.startRow, // endRow - startRow gives perPage value
                        offset: params.request.startRow,
                    })
                        .pipe(
                            takeWhile(_ => _that.alive),
                        )
                        .subscribe((res: any) => {
                            // apply sort and filter
                            const dataAfterSortingAndFiltering = sortAndFilter(
                                res.Data,
                                params.request.sortModel,
                                params.request.filterModel,
                            );

                            // create and append grid actions to api result
                            const rowsThisPage = _that.addActions(dataAfterSortingAndFiltering);
                            let lastRow = -1;
                            if (dataAfterSortingAndFiltering.length <= params.request.endRow) {
                                lastRow = dataAfterSortingAndFiltering.length;
                            }
                            params.successCallback(rowsThisPage, lastRow);
                        });

                }, 500);
            }
        };
        this.gridOptions.api.setServerSideDatasource(dataSource);
    }
```
