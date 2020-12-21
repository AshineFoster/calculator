<script>
	import currency from "currency.js";
	import "../node_modules/uikit/dist/css/uikit.min.css";
	import "../node_modules/uikit/dist/js/uikit.min.js";
	import "../node_modules/uikit/dist/js/uikit-icons.min.js";

	const cess = 0.0033;
	const trade = 0.0;
	const gct = 0.15;
	const minShares = 1;

	let buyOrSell = "buy",
		sharesOrPrice = "price",
		numOfShares,
		pricePerShare,
		commissionPercent = 2,
		minComm = 0;

	// variables when calculating # of shares
	let buyOrSellShares = "buy",
		avaliableCash,
		pricePerShareShares,
		commissionPercentShares = 2,
		minCommShares = 0;

	$: priceDisplay = findPrice(
		buyOrSell,
		numOfShares,
		pricePerShare,
		commissionPercent,
		minComm
	);

	$: sharesDisplay = calcMaxShares(
		buyOrSellShares,
		avaliableCash,
		pricePerShareShares,
		commissionPercentShares,
		minCommShares
	);

	// find out how much money it cost to certain trades
	let findPrice = function (
		buyOrSell,
		numOfShares,
		pricePerShare,
		commissionPercent,
		minComm
	) {
		let grossPrice = currency(numOfShares).multiply(pricePerShare),
			minCommission = currency(minComm),
			commission = currency(commissionPercent, { precision: 6 }).divide(
				100
			),
			tempCommission = currency(grossPrice.multiply(commission)),
			finalCommission = currency(
				tempCommission.intValue < minCommission.intValue
					? minCommission
					: tempCommission
			),
			cessFee = grossPrice.multiply(cess),
			tradeFee = grossPrice.multiply(trade),
			gctFee = finalCommission.add(cessFee).add(tradeFee).multiply(gct),
			totalFees = gctFee.add(cessFee).add(tradeFee).add(finalCommission),
			finalPrice = currency(0);

		if (buyOrSell === "buy") {
			finalPrice = grossPrice.add(totalFees);
		} else if (buyOrSell === "sell") {
			finalPrice = grossPrice.subtract(totalFees);
		}

		return {
			grossPrice: grossPrice,
			finalCommission: finalCommission,
			cess: cessFee,
			trade: tradeFee,
			gct: gctFee,
			totalFees: totalFees,
			finalPrice: finalPrice,
		};
	};

	// calculate number of shares that can be traded
	let calcMaxShares = function (
		buyOrSell,
		avaliableCash,
		pricePerShare,
		commissionPercent,
		minComm
	) {
		let cash = currency(avaliableCash),
			tempMaxShares = pricePerShare
				? Math.ceil(cash.divide(pricePerShare).value)
				: currency(1),
			maxShares = buyOrSell === "buy" ? tempMaxShares : tempMaxShares * 2,
			minValues = findPrice(
				buyOrSell,
				minShares,
				pricePerShare,
				commissionPercent,
				minComm
			),
			ans = iterativeFunction(
				maxShares,
				minShares,
				buyOrSell,
				cash,
				pricePerShare,
				commissionPercent,
				minComm
			);
		return ans;
	};

	// Iterative function to implement Binary Search
	let iterativeFunction = function (
		maxShares,
		minShares,
		buyOrSell,
		cash,
		pricePerShare,
		commissionPercent,
		minComm
	) {
		while (minShares <= maxShares) {
			let mid = Math.floor((minShares + maxShares) / 2);

			let midPrice1 = findPrice(
				buyOrSell,
				mid,
				pricePerShare,
				commissionPercent,
				minComm
			);
			let midPrice2 = findPrice(
				buyOrSell,
				mid + 1,
				pricePerShare,
				commissionPercent,
				minComm
			);

			if (midPrice1.finalPrice.intValue === cash.intValue) {
				midPrice1.sharesNum = mid;
				return midPrice1;
			} else if (
				midPrice1.finalPrice.intValue < cash.intValue &&
				midPrice2.finalPrice.intValue > cash.intValue
			) {
				if (buyOrSell === "buy") {
					midPrice1.sharesNum = mid;
					return midPrice1;
				} else {
					midPrice2.sharesNum = mid + 1;
					return midPrice2;
				}
			} else if (midPrice1.finalPrice.intValue > cash.intValue)
				maxShares = mid - 1;
			else minShares = mid + 1;
		}

		return false;
	};

	// use console to test values
	// let v = findPrice("buy", 1000, 40, 0.5, 0);
	// console.log(v);
	// let w = calcMaxShares("buy", 40381.8, 40, 0.5, 0);
	// console.log(w);

	// let vs = findPrice("sell", 961, 43.5, 0.75, 0);
	// console.log(vs);
	// let ws = calcMaxShares("sell", 41284.3, 43.5, 0.75, 0);
	// console.log(ws);
</script>

<div class="uk-container uk-container-large">

	<form class="uk-form-horizontal">
		
		<div class="uk-margin">
			<div class="uk-form-label">Price / Shares:</div>
			<div class="uk-form-controls uk-form-controls-text">
				<label>
					<input
						class="uk-radio"
						type="radio"
						bind:group={sharesOrPrice}
						value={'price'} />
					Price
				</label>
				<label>
					<input
						class="uk-radio"
						type="radio"
						bind:group={sharesOrPrice}
						value={'shares'} />
					Shares
				</label>
			</div>
		</div>
	</form>

	{#if sharesOrPrice === 'price'}
		<form class="uk-form-horizontal">
			<div class="uk-margin">
				<div class="uk-form-label">Buy / Sell:</div>
				<div class="uk-form-controls uk-form-controls-text">
					<label>
						<input
							class="uk-radio"
							type="radio"
							bind:group={buyOrSell}
							value={'buy'} />
						Buy
					</label>
					<label>
						<input
							class="uk-radio"
							type="radio"
							bind:group={buyOrSell}
							value={'sell'} />
						Sell</label>
				</div>
			</div>

			<div class="uk-margin">
				<label class="uk-form-label" for="num-of-shares">Number of
					shares:</label>
				<div class="uk-form-controls">
					<input
						class="uk-input"
						id="num-of-shares"
						type="number"
						bind:value={numOfShares}
						min="0" />
				</div>
			</div>

			<div class="uk-margin">
				<label class="uk-form-label" for="price-per-share-price">Price
					per share:</label>
				<div class="uk-form-controls">
					<input
						class="uk-input"
						id="price-per-share-price"
						type="number"
						bind:value={pricePerShare}
						min="0"
						step="0.01" />
				</div>
			</div>

			<div class="uk-margin">
				<label class="uk-form-label" for="commission-percent-price">
					Broker commission (%):</label>
				<div class="uk-form-controls">
					<input
						class="uk-input"
						id="commission-percent-price"
						type="number"
						bind:value={commissionPercent}
						min="0"
						step="0.01" />
				</div>
			</div>
		</form>

		<table class="uk-table uk-table-divider">
			<tr>
				<td>Gross price</td>
				<td>{priceDisplay.grossPrice.format()}</td>
			</tr>
			<tr>
				<td>Commission</td>
				<td>{priceDisplay.finalCommission.format()}</td>
			</tr>
			<tr>
				<td>Cess fee</td>
				<td>{priceDisplay.cess.format()}</td>
			</tr>
			<tr>
				<td>Trade fee</td>
				<td>{priceDisplay.trade.format()}</td>
			</tr>
			<tr>
				<td>GCT tax</td>
				<td>{priceDisplay.gct.format()}</td>
			</tr>
			<tr>
				<td>Total fees</td>
				<td>{priceDisplay.totalFees.format()}</td>
			</tr>
			<tr>
				<td>Final price</td>
				<td>{priceDisplay.finalPrice.format()}</td>
			</tr>
		</table>
	{:else if sharesOrPrice === 'shares'}
		<form class="uk-form-horizontal">
			<div class="uk-margin">
				<div class="uk-form-label">Buy / Sell:</div>
				<div class="uk-form-controls uk-form-controls-text">
					<label>
						<input
							class="uk-radio"
							type="radio"
							bind:group={buyOrSellShares}
							value={'buy'} />
						Buy
					</label>
					<label>
						<input
							class="uk-radio"
							type="radio"
							bind:group={buyOrSellShares}
							value={'sell'} />
						Sell</label>
				</div>
			</div>

			<div class="uk-margin">
				<label class="uk-form-label" for="cash-avaliable">Avaliable
					cash:</label>
				<div class="uk-form-controls">
					<input
						class="uk-input"
						id="cash-avaliable"
						type="number"
						bind:value={avaliableCash}
						step="0.01"
						min="0" />
				</div>
			</div>

			<div class="uk-margin">
				<label class="uk-form-label" for="price-per-share-price">Price
					per share:</label>
				<div class="uk-form-controls">
					<input
						class="uk-input"
						id="price-per-share-price"
						type="number"
						bind:value={pricePerShareShares}
						min="0"
						step="0.01" />
				</div>
			</div>

			<div class="uk-margin">
				<label class="uk-form-label" for="commission-percent-price">
					Broker commission (%):</label>
				<div class="uk-form-controls">
					<input
						class="uk-input"
						id="commission-percent-price"
						type="number"
						bind:value={commissionPercentShares}
						min="0"
						step="0.01" />
				</div>
			</div>
		</form>

		{#if avaliableCash && pricePerShareShares && avaliableCash > pricePerShareShares}
			<!-- content here -->
			<table class="uk-table  uk-table-divider">
				<tr>
					<td>Number of shares</td>
					<td>
						{new Intl.NumberFormat('en-JM').format(sharesDisplay.sharesNum)}
					</td>
				</tr>

				<tr>
					<td>Gross price</td>
					<td>{sharesDisplay.grossPrice.format()}</td>
				</tr>
				<tr>
					<td>Commission</td>
					<td>{sharesDisplay.finalCommission.format()}</td>
				</tr>
				<tr>
					<td>Cess fee</td>
					<td>{sharesDisplay.cess.format()}</td>
				</tr>
				<tr>
					<td>Trade fee</td>
					<td>{sharesDisplay.trade.format()}</td>
				</tr>
				<tr>
					<td>GCT tax</td>
					<td>{sharesDisplay.gct.format()}</td>
				</tr>
				<tr>
					<td>Total fees</td>
					<td>{sharesDisplay.totalFees.format()}</td>
				</tr>
				<tr>
					<td>Final price</td>
					<td>{sharesDisplay.finalPrice.format()}</td>
				</tr>
			</table>
		{:else}
			<!-- else content here -->
			<p>enter info</p>
		{/if}
	{:else}
		<p>should not show</p>
	{/if}
</div>
