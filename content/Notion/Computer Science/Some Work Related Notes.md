We are looking to aggregate rows in the table where the `deal_id` is the same. The following columns are easily merged into one:

- `deal.tradedate`
- `deal.dealid`
- `deal.status`
- `deal.book_date`
- `deal.excutedtime`
- `deal.trdcurrency`
- `deal.version`
- `instr.ticker`
- `instr.exchcode`
- `instr.type`
- `deal.executorid`
- `deal.createuserid`
- `deal.createtimeutc`
- `bk.bkname`, `deal.capacity`, `deal.qty`, `deal.record`, `deal.trdprice`, `deal.trdval`

The ones we need to think more about are `bk.bkname`, `deal.capacity`, `deal.qty`, `deal.record`, `deal.trdprice`, `deal.trdval`