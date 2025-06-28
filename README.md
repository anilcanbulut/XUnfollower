# XUnfollower

This repository automates the process of unfollowing accounts on your X account that do not follow you back. It offers a straightforward solution for managing your follow list. If you encounter any issues, please open an issue in the repository.

Note: All features have been tested in a desktop environment using the Chrome browser. It is recommended to use the same setup, although it may also work in other browsers.

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
