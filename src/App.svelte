<script>
	import currency from "currency.js";

	const cess = 0.0033;
	const trade = 0.0;
	const gct = 0.15;
	const minShares = 1;

	const USD = (value) => currency(value, { symbol: "$", precision: 4 });
	const JMD = (value) => currency(value, { symbol: "$", precision: 2 });

	let sharesOrPrice = "price",
		currencyType = "JMD",
		buyOrSell = "buy",
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

	$: minValueShares = findPrice(
		buyOrSellShares,
		minShares,
		pricePerShareShares,
		commissionPercentShares,
		minCommShares
	);

	$: currFormat = function (value, currency) {
		let formattedCurrency = "";
		if (currency == "JMD") {
			formattedCurrency = JMD(value).format();
		} else {
			formattedCurrency = USD(value).format();
		}
		return formattedCurrency;
	};

	$: presForm = function (currency) {
		let prec = "";
		if (currency == "JMD") {
			prec = "0.01";
		} else {
			prec = "0.0001";
		}
		return prec;
	};

	// find out how much money it cost to buy/sell a certain number of shares
	let findPrice = function (
		buyOrSell = "buy",
		numOfShares = 0,
		pricePerShare = 0,
		commissionPercent,
		minComm
	) {
		let curren = (value) => currency(value, { precision: 6 });
		let grossPrice = curren(numOfShares).multiply(pricePerShare),
			minCommission = curren(minComm),
			commission = curren(commissionPercent, { precision: 6 }).divide(
				100
			),
			tempCommission = curren(grossPrice.multiply(commission)),
			finalCommission = curren(
				tempCommission.intValue < minCommission.intValue
					? minCommission
					: tempCommission
			),
			cessFee = grossPrice.multiply(cess),
			tradeFee = grossPrice.multiply(trade),
			gctFee = finalCommission.add(cessFee).add(tradeFee).multiply(gct),
			totalFees = gctFee.add(cessFee).add(tradeFee).add(finalCommission),
			finalPrice = curren(0);

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
		let curren = (value) => currency(value, { precision: 6 });
		let cash = curren(avaliableCash),
			tempMaxShares = pricePerShare
				? Math.ceil(cash.divide(pricePerShare).value)
				: curren(1),
			maxShares = buyOrSell === "buy" ? tempMaxShares : tempMaxShares * 2,
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
</script>

<div class="uk-container uk-container-large">
	<div class="uk-grid-divider" uk-grid>
		<div class="uk-width-1-2 uk-grid-item-match">
			<form class="uk-form-horizontal">
				<div class="uk-margin">
					<div class="uk-form-label">Price / Shares:</div>
					<div class="uk-form-controls uk-form-controls-text">
						<label>
							<input
								class="uk-radio"
								type="radio"
								bind:group={sharesOrPrice}
								value={"price"}
							/>
							Price
						</label>
						<br />
						<label>
							<input
								class="uk-radio"
								type="radio"
								bind:group={sharesOrPrice}
								value={"shares"}
							/>
							Shares
						</label>
					</div>

					<div class="uk-form-label">Currency:</div>
					<div class="uk-form-controls uk-form-controls-text">
						<label>
							<input
								class="uk-radio"
								type="radio"
								bind:group={currencyType}
								value={"JMD"}
							/>
							JMD
						</label>
						<br />
						<label>
							<input
								class="uk-radio"
								type="radio"
								bind:group={currencyType}
								value={"USD"}
							/>
							USD
						</label>
					</div>
				</div>
			</form>

			{#if sharesOrPrice === "price"}
				<form class="uk-form-horizontal">
					<div class="uk-margin">
						<div class="uk-form-label">Buy / Sell:</div>
						<div class="uk-form-controls uk-form-controls-text">
							<label>
								<input
									class="uk-radio"
									type="radio"
									bind:group={buyOrSell}
									value={"buy"}
								/>
								Buy
							</label>
							<br />
							<label>
								<input
									class="uk-radio"
									type="radio"
									bind:group={buyOrSell}
									value={"sell"}
								/>
								Sell</label
							>
						</div>
					</div>

					<div class="uk-margin">
						<label class="uk-form-label" for="num-of-shares"
							>Number of shares:</label
						>
						<div class="uk-form-controls">
							<input
								class="uk-input"
								id="num-of-shares"
								type="number"
								bind:value={numOfShares}
								min="0"
							/>
						</div>
					</div>

					<div class="uk-margin">
						<label class="uk-form-label" for="price-per-share-price"
							>Price per share:</label
						>
						<div class="uk-form-controls">
							<input
								class="uk-input"
								id="price-per-share-price"
								type="number"
								bind:value={pricePerShare}
								min="0"
								step={presForm(currencyType)}
							/>
						</div>
					</div>

					<div class="uk-margin">
						<label
							class="uk-form-label"
							for="commission-percent-price"
						>
							Broker commission (%):</label
						>
						<div class="uk-form-controls">
							<input
								class="uk-input"
								id="commission-percent-price"
								type="number"
								bind:value={commissionPercent}
								min="0"
								step="0.01"
							/>
						</div>
					</div>
				</form>

			{:else if sharesOrPrice === "shares"}
				<form class="uk-form-horizontal">
					<div class="uk-margin">
						<div class="uk-form-label">Buy / Sell:</div>
						<div class="uk-form-controls uk-form-controls-text">
							<label>
								<input
									class="uk-radio"
									type="radio"
									bind:group={buyOrSellShares}
									value={"buy"}
								/>
								Buy
							</label>
							<br />
							<label>
								<input
									class="uk-radio"
									type="radio"
									bind:group={buyOrSellShares}
									value={"sell"}
								/>
								Sell</label
							>
						</div>
					</div>

					<div class="uk-margin">
						<label class="uk-form-label" for="cash-avaliable"
							>Avaliable cash:</label
						>
						<div class="uk-form-controls">
							<input
								class="uk-input"
								id="cash-avaliable"
								type="number"
								bind:value={avaliableCash}
								min="0"
								step={presForm(currencyType)}
							/>
						</div>
					</div>

					<div class="uk-margin">
						<label class="uk-form-label" for="price-per-share-price"
							>Price per share:</label
						>
						<div class="uk-form-controls">
							<input
								class="uk-input"
								id="price-per-share-price"
								type="number"
								bind:value={pricePerShareShares}
								min="0"
								step={presForm(currencyType)}
							/>
						</div>
					</div>

					<div class="uk-margin">
						<label
							class="uk-form-label"
							for="commission-percent-price"
						>
							Broker commission (%):</label
						>
						<div class="uk-form-controls">
							<input
								class="uk-input"
								id="commission-percent-price"
								type="number"
								bind:value={commissionPercentShares}
								min="0"
								step="0.01"
							/>
						</div>
					</div>
				</form>
			{:else}
				<p>should not show</p>
			{/if}
		</div>

		<div class="uk-width-1-2">
			{#if sharesOrPrice === "price"}
				<table class="uk-table uk-table-divider uk-table-middle">
					<tr>
						<td>Gross price</td>
						<td class="uk-text-right"
							>{currFormat(
								priceDisplay.grossPrice,
								currencyType
							)}</td
						>
					</tr>
					<tr>
						<td>Commission</td>
						<td class="uk-text-right"
							>{currFormat(
								priceDisplay.finalCommission,
								currencyType
							)}</td
						>
					</tr>
					<tr>
						<td>Cess fee</td>
						<td class="uk-text-right"
							>{currFormat(priceDisplay.cess, currencyType)}</td
						>
					</tr>
					<tr>
						<td>Trade fee</td>
						<td class="uk-text-right"
							>{currFormat(priceDisplay.trade, currencyType)}</td
						>
					</tr>
					<tr>
						<td>GCT tax</td>
						<td class="uk-text-right"
							>{currFormat(priceDisplay.gct, currencyType)}</td
						>
					</tr>
					<tr>
						<td>Total fees</td>
						<td class="uk-text-right"
							>{currFormat(
								priceDisplay.totalFees,
								currencyType
							)}</td
						>
					</tr>
					<tr>
						<td>Final price</td>
						<td class="uk-text-right"
							>{currFormat(
								priceDisplay.finalPrice,
								currencyType
							)}</td
						>
					</tr>
				</table>
			{:else if sharesOrPrice === "shares"}
				{#if avaliableCash && pricePerShareShares && avaliableCash >= minValueShares.finalPrice.value}
					<!-- content here -->
					<table class="uk-table  uk-table-divider uk-table-middle">
						<tr>
							<td>Number of shares</td>
							<td class="uk-text-right">
								{new Intl.NumberFormat("en-JM").format(
									sharesDisplay.sharesNum
								)}
							</td>
						</tr>

						<tr>
							<td>Gross price</td>
							<td class="uk-text-right"
								>{currFormat(
									sharesDisplay.grossPrice,
									currencyType
								)}</td
							>
						</tr>
						<tr>
							<td>Commission</td>
							<td class="uk-text-right"
								>{currFormat(
									sharesDisplay.finalCommission,
									currencyType
								)}</td
							>
						</tr>
						<tr>
							<td>Cess fee</td>
							<td class="uk-text-right"
								>{currFormat(
									sharesDisplay.cess,
									currencyType
								)}</td
							>
						</tr>
						<!-- <tr>
							<td>Trade fee</td>
							<td class="uk-text-right"
								>{currFormat(
									sharesDisplay.trade,
									currencyType
								)}</td
							>
						</tr> -->
						<tr>
							<td>GCT tax</td>
							<td class="uk-text-right"
								>{currFormat(
									sharesDisplay.gct,
									currencyType
								)}</td
							>
						</tr>
						<tr>
							<td>Total fees</td>
							<td class="uk-text-right"
								>{currFormat(
									sharesDisplay.totalFees,
									currencyType
								)}</td
							>
						</tr>
						<tr>
							<td>Final price</td>
							<td class="uk-text-right"
								>{currFormat(
									sharesDisplay.finalPrice,
									currencyType
								)}</td
							>
						</tr>
					</table>
				{:else}
					<!-- else content here -->
					<p>
						Not enough cash.
						{currFormat(minValueShares.finalPrice, currencyType)}
						is needed to {buyOrSellShares}
						{minShares}
						share.
					</p>
				{/if}
			{/if}
		</div>
	</div>
</div>
