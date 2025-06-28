# XUnfollower

This repository is reponsible for unfollowing the accounts in your X account that did not follow you back. This repository provides a simple yet effective solution to the X unfollowing problem. If you see a problem, just create an issue.

Note: All the things are tested on Desktop environment in Chrome Browser. So it is highly recommended to follow this environment setup even though it might work in other browsers as well.

## Usage
The way how the XUnfollower works is very simple. Here are the steps you need to follow:

1. First, you should sign in to your X account and go into the "Following" page where the accounts you follow are listed.
2. Right click on your browser and click to "Inspect" to open the DevTools.
3. Click on "Console" tab.
4. In the console tab, paste the code below to the new code line starts with ">" sign and enter. After you click enter the code will start and you will see it will sequentially start to unfollow the acounts that do not follow you.
'''bash
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
'''
