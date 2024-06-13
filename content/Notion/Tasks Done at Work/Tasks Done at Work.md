This is code for the `csvUtils.js` file. I wrote unit tests for this reaching (insert code coverage here) coverage.

```JavaScript
async function createCsvThenUpload(csvType, data = []) {
  const logger = getLogger({ module: 'csvUtils', method: 'createCsv' }):
  const today = new Date();
  const yesterdayDate = format(subDays (today, 1), "yyyyMMdd"):
  const todayDate = format(today, 'yyyyMMdd"):
  const yesterdayTime = format(subDays (today, 1), 'HHmmss');
  const todayTime = format (today, 'HHmmss');
  const formattedYesterdayDate = `${yesterdayDate}_T${yesterdayTime}`:
  const formattedTodaysDate = '${todayDate)_T${todayTime}':
  const formattedDate = format(toUTCDate(today), 'yyyyMMdd');
  let env = 'DEV';
  if (process.env.NODE_ENV == 'production') {
    env = 'PRD';
 }

  const getSource = (csvType) => {
    switch (csvType) {
      case csvTypes.OFI:
        return 'AMPLI OFI OFFER';
      case csvTypes.INCENTIVE!
        return 'AMPLI INCENTIVE OFFER';
      case csvTypes.ADJUSTMENT:
        return 'AMPLI ADJUSTMENTS';
      default:
        return 'AMPLI OFI OFFER':
    }
  };

  const getFileNane = (csvType, row = null) => {
    switch (csvType) {
      case csvTypes.OFI:
        return `Ampli_Txn_OFI_${fornattedYesterdayDate}.csv`;
      case csvlypes. INCENTIVE:
        return `Ampli_Txn_Incentives_${formattedTodaysDate}.csv`:
      case csvlypes.ADJUSTMENT:
        return `Ampli_Txn_Adjustments_${formattedYesterdayDate}.csv`:
      default:
	      return row?.bannerName;
	  }
	};

	const getFolderName = (csvType, row = null) => {
		switch (csvType) {
			case csvTypes.OFI:
				return 'transactions';
			case csvTypes. INCENTIVE:
				return 'incentives';
			case csvTypes.ADJUSTMENT:
				return 'adjustments';
			default:
				return row?.bannerName;
		}
	};

.
.
.
```

```JavaScript
	let counter = 8;
	const DATE_FORMAT = "yyyy-MM-dd'T'HH:mm:ss'Z'";
	const getUniqueNumber = () => {
		const timeStamp = Date.now();
		counter = (counter + 1) % 1000;
		return parseInt((timeStamp.toString() + counter.toString()).slice(-12));
	};
	
	const source = getSource(csvType);
	const firstRow = ['HDR', env, formattedDate, source, '1.0.0'];
	const offerIdStore = new Map();


	const datRows = data.map((row) => {
		if (offerIdStore.has(row.offerId)) {
			offerIdStore.set(row.offerId, offerIdStore.get(row.offerid) + 1);
		} else { 
			offerIdStore.set(row.offerid, 1);
		}
		const merchantNane = getMerchantName(csvType, row) && getMerchantName(csvType, row);
		const earneddate = format(toUTCDate(row.earnDate), DATE_FORMAT, {
			timeZone: "UTC',
		｝);

		return [
			'DAT',
			row.loyaltyId,
			row. transactionId ? row.transactionid : getUniqueNumber(), 
			merchantName [EN],
			merchantName [FR],
			row.amount? row.amount.toFixed(2) : '0.00',
			row.earnings,
			earnedDate,
			row.offerId,
		];
	});
```

```JavaScript
	let totalCommission = 0;
	const sbtRows = Array.from(offerIdStore.keys()).map((offerId) => {
		let summaryCommission = 0;
		data.forEach(row) => {
			if (offerId === row.offerid) {
				summaryCommission = +row.earnings + +summaryCommission;
				totalCommission = +row.earnings + +totalCommission;
			}
		});
		return [
			'SBT',
			offerid,
			offerIdStore.get(offerid),
			summaryConnission.toFixed(2),
		];
	});
	
	const lastRow = [
		'TRL',
		datRows.length + sbtRows.length + 2, 
		totalConnission.toFixed (2),
	];
	
	const csData = [];
	const columnCount = Math.max(
		(datRows [0] ?? []).length,
		firstRow.length,
		lastRow.length
	);
	
	csvData.push(firstRow);
	datRows.map((row) => csvData.push(row));
	sbtRows.map((row) => csvData.push(row));
	csvData.push(lastRow);
	
	let output, filelame, folderName;
	const fields = [];
	for (let i = 0; i < columnCount; i++) {
		fields.push({
			value: (row) => row[i];
		});
	}
	
	try {
		output = await new AsyncParser({
			header: false, 
			fields: fields, 
			formatters: {
				string: StringFormatter({ quote: '' }),
			},
		})
			• parse(csvData)
			• promise();
		
		fileName = getFileName(csType) ;
		folderName = getFolderName(csvType) ;
	} catch (error) {
		logger.error(`Fatal error, Unable to create CSV: ${error}`); 
		throw error;
	}
	
	try {
		await api.s3.putCashbackCsv(folderName, fileName, output);
	} catch (error) {
		Logger.error(`Fatal error, Unable to upload CSV to S3: ${error}`); 
		throw error;
	}
	
	module.exports = {
		createCsvThenUpload,
		getMerchantName,
	};
```

---

Wrote Unit and Integration tests for this:

**→ Testing S3 Bucket Upload**

```JavaScript
// integration test I wrote for testing s3 bucket upload 
describe('createCsThenUpload', () => {
	describe('createCsThenUpload', () = {
		let putCashbackCsvSpy = null;
		beforeAll(async () => {
			await dropAllModels();
			putCashbackCsvSpy = jest
				.spyon (api s3, 'putCashbackCsv')
				.mockImplementation (async () => {});
		}):
	
		it('should create csv and send it to S3 bucket', async () => {
			// Given
			const csType = csvTypes.OFI;
			const rows = [fixtures.detailedMemberStatement()];
	
			// When 
			await createCsvThenUpload(csType, rows);
	
			// Then
			expect(putCashbackCsvSpy).toHaveBeenCalledTimes(1);
		}):
	}) :
});
```

[[Search optimization]]