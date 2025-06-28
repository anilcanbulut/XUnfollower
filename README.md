# XUnfollower

This repository automates the process of unfollowing accounts on your X account that do not follow you back. It offers a straightforward solution for managing your follow list. If you encounter any issues, please open an issue in the repository.

Note-1: All features have been tested in a desktop environment using the Chrome browser. It is recommended to use the same setup, although it may also work in other browsers.
Note-2: This script will unfollow all accounts that do not follow you back. This includes popular or high-profile accounts that you may not expect to follow you. Be sure you are okay with this before running the script.

## Usage

Using XUnfollower is straightforward. Follow these steps:

1. Sign in to your X account and navigate to the **Following** page.
2. Right-click anywhere on the page and select **Inspect** to open the Developer Tools.
3. Click the **Console** tab.
4. Paste the script below into the command line that starts with the `>` symbol, then press **Enter**.

The script will begin running and will automatically unfollow accounts that do not follow you back, one by one.

```javascript
	(async function autoUnfollowLoop() {
	  const sleep = (ms) => new Promise(resolve => setTimeout(resolve, ms));
	  const randomDelay = () => Math.floor(200 + Math.random() * 800); // 200–1000 ms

	  let totalUnfollowed = 0;
	  let runCount = 1;

	  while (true) {
		console.log(`\n🔁 Run #${runCount} starting...`);

		const userCells = [...document.querySelectorAll('button[data-testid="UserCell"]')];
		console.log(`Detected ${userCells.length} visible accounts.`);

		let unfollowedThisRound = 0;

		for (let cell of userCells) {
		  const handle = cell.querySelector('div[dir="ltr"] span')?.innerText || "unknown";

		  const spans = [...cell.querySelectorAll('span')].map(s => s.textContent);
		  const followsYou = spans.includes("Follows you");

		  if (!followsYou) {
			console.log(`➡️ Unfollowing @${handle}...`);

			const followingBtn = [...cell.querySelectorAll('button[role="button"]')]
			  .find(btn => btn.innerText.trim() === "Following");

			if (followingBtn) {
			  followingBtn.click();
			  await sleep(500);

			  const confirmBtn = document.querySelector('button[data-testid="confirmationSheetConfirm"]');
			  if (confirmBtn) {
				confirmBtn.click();
				unfollowedThisRound++;
				totalUnfollowed++;
				await sleep(1000);
			  } else {
				console.log(`⚠️ Confirmation button not found for @${handle}`);
			  }
			} else {
			  console.log(`⚠️ 'Following' button not found for @${handle}`);
			}

			await sleep(randomDelay());
		  } else {
			console.log(`✅ Keeping @${handle} (follows you)`);
		  }
		}

		console.log(`🔁 Run #${runCount} complete. Unfollowed ${unfollowedThisRound} accounts.`);

		if (unfollowedThisRound === 0) {
		  console.log(`✅ All done. Total unfollowed: ${totalUnfollowed}. No more to unfollow.`);
		  break;
		}

		runCount++;
		await sleep(1500); // Small delay between loops
	  }
	})();
```

## Update
Here is another script that this time it asks you to pass a threshold for the number of followers for each account. If the account's number of followers is greater than or equal to the threshold you passed then the account is skipped and not unfollowed.

```javascript
	(async function autoUnfollowLoopWithInputs() {
	  const sleep = (ms) => new Promise(resolve => setTimeout(resolve, ms));
	  const randomDelay = () => Math.floor(200 + Math.random() * 800); // 200–1000 ms
	  const randomHoverDelay = () => Math.floor(1500 + Math.random() * 1500); // 1500–3000 ms
	  const MAX_HOVER_ATTEMPTS = 5;

	  // --- User Inputs ---
	  const input = prompt("Enter minimum follower count to skip unfollowing (e.g. 10000):");
	  const ACCOUNT_FOLLOWER_COUNT = parseInt(input?.trim());

	  if (isNaN(ACCOUNT_FOLLOWER_COUNT)) {
		console.log("❌ Invalid follower count. Exiting.");
		return;
	  }

	  prompt(`Threshold set to ${ACCOUNT_FOLLOWER_COUNT}. Press OK or Enter to start...`);

	  let totalUnfollowed = 0;
	  let runCount = 1;

	  function extractFollowerCountFromCard(cardElement) {
		const link = cardElement.querySelector('a[href$="/verified_followers"]');
		if (!link) return null;

		const spans = link.querySelectorAll('span');
		for (let i = 0; i < spans.length - 1; i++) {
		  const label = spans[i + 1].innerText.trim().toLowerCase();
		  if (label === "followers") {
			let raw = spans[i].innerText.trim().toUpperCase().replace(/\s/g, '');
			try {
			  if (raw.endsWith("K")) return Math.round(parseFloat(raw) * 1000);
			  if (raw.endsWith("M")) return Math.round(parseFloat(raw) * 1000000);
			  if (raw.endsWith("B")) return Math.round(parseFloat(raw) * 1000000000);
			  return parseInt(raw.replace(/,/g, '').replace(/\./g, ''));
			} catch {
			  return null;
			}
		  }
		}
		return null;
	  }

	  while (true) {
		console.log(`\n🔁 Run #${runCount} starting...`);

		const userCells = [...document.querySelectorAll('button[data-testid="UserCell"]')];
		console.log(`Detected ${userCells.length} visible accounts.`);

		let unfollowedThisRound = 0;

		for (let cell of userCells) {
		  const handle = cell.querySelector('div[dir="ltr"] span')?.innerText || "unknown";
		  const spans = [...cell.querySelectorAll('span')].map(s => s.textContent);
		  const followsYou = spans.includes("Follows you");

		  if (followsYou) {
			console.log(`✅ Keeping @${handle} (follows you)`);
			continue;
		  }

		  const usernameLink = cell.querySelector('a[href^="/"]');
		  if (!usernameLink) {
			console.log(`⚠️ Skipping @${handle} — no profile link`);
			continue;
		  }

		  usernameLink.scrollIntoView({ behavior: "smooth", block: "center" });

		  let followerCount = null;
		  let hoverCard = null;
		  let attempts = 0;

		  while (attempts < MAX_HOVER_ATTEMPTS && !hoverCard) {
			const beforeDivs = new Set([...document.querySelectorAll('div')]);

			usernameLink.dispatchEvent(new MouseEvent('mouseout', { bubbles: true }));
			await sleep(100);
			usernameLink.dispatchEvent(new MouseEvent('mouseover', { bubbles: true }));
			await sleep(randomHoverDelay()); // 1.5–3s

			const afterDivs = [...document.querySelectorAll('div')];
			const newDivs = afterDivs.filter(div => !beforeDivs.has(div));
			hoverCard = newDivs.find(div => div.innerText.includes("Followers"));

			if (hoverCard) {
			  followerCount = extractFollowerCountFromCard(hoverCard);
			}

			attempts++;
		  }

		  if (followerCount === null) {
			console.log(`⚠️ Skipping @${handle} — follower count not found after ${MAX_HOVER_ATTEMPTS} attempts`);
			continue;
		  }

		  if (followerCount >= ACCOUNT_FOLLOWER_COUNT) {
			console.log(`🚫 Skipping @${handle} — has ${followerCount} followers (≥ threshold)`);
			continue;
		  }

		  console.log(`➡️ Unfollowing @${handle} — ${followerCount} followers`);

		  const followingBtn = [...cell.querySelectorAll('button[role="button"]')]
			.find(btn => btn.innerText.trim() === "Following");

		  if (followingBtn) {
			followingBtn.click();
			await sleep(500);

			const confirmBtn = document.querySelector('button[data-testid="confirmationSheetConfirm"]');
			if (confirmBtn) {
			  confirmBtn.click();
			  unfollowedThisRound++;
			  totalUnfollowed++;
			  await sleep(1000);
			} else {
			  console.log(`⚠️ Confirmation button not found for @${handle}`);
			}
		  } else {
			console.log(`⚠️ 'Following' button not found for @${handle}`);
		  }

		  await sleep(randomDelay());
		}

		console.log(`🔁 Run #${runCount} complete. Unfollowed ${unfollowedThisRound} accounts.`);

		if (unfollowedThisRound === 0) {
		  console.log(`✅ All done. Total unfollowed: ${totalUnfollowed}. No more to unfollow.`);
		  break;
		}

		runCount++;
		await sleep(1500);
	  }
	})();

```