---
layout: post
hidden: false
title: Setting Up ServerSideDatasource for agGrid Angular with Pagination
tags: []
---
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
