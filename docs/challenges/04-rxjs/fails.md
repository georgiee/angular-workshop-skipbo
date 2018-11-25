## fil 01
this will accumulate susbcriptions to the nextTurn
remvoe and app players, and you will get many stream values form non existing players.

   merge(this._game.players$)
    .pipe(
      mergeMap(value => value),
      mergeMap(player => {
        console.log('merge map called')
        return player.nextTurn.pipe(tap(_ => console.log('next turn arrvied')));
      }),
      tap(value => console.log('value: ', value))
    )
    .subscribe();
