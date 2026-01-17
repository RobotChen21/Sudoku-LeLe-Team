<script>
	import { candidates } from '@sudoku/stores/candidates';
	import { userGrid } from '@sudoku/stores/grid';
	import { cursor } from '@sudoku/stores/cursor';
	import { hints } from '@sudoku/stores/hints';
	import { notes } from '@sudoku/stores/notes';
	import { settings } from '@sudoku/stores/settings';
	import { keyboardDisabled } from '@sudoku/stores/keyboard';
	import { gamePaused } from '@sudoku/stores/game';

	import { get } from 'svelte/store';
	let memorySave = null;

	$: hintsAvailable = $hints > 0;

	function handleHint() {
		if (hintsAvailable) {
			if ($candidates.hasOwnProperty($cursor.x + ',' + $cursor.y)) {
				candidates.clear($cursor);
			}

			userGrid.applyHint($cursor);
		}
	}

	function saveSnapshot() {
		// 保存当前的Grid数据（深拷贝）
		const currentGrid = get(userGrid);
		const currentCandidates = get(candidates);
		const currentNotes = get(notes);
		
		// 深拷贝网格数据
		const savedGrid = JSON.parse(JSON.stringify(currentGrid));
		const savedCandidates = JSON.parse(JSON.stringify(currentCandidates));
		
		// 保存到memorySave中
		memorySave = {
			grid: savedGrid,
			candidates: savedCandidates,
			notes: currentNotes
		};
		
    	console.log('已保存当前Grid数据:', memorySave);
  	}

	function loadSnapshot() {
		// 恢复之前保存的Grid数据
		if (memorySave) {
			// 恢复网格数据 - 使用现有API逐个单元格恢复
			const savedGrid = JSON.parse(JSON.stringify(memorySave.grid));
			const savedCandidates = JSON.parse(JSON.stringify(memorySave.candidates));
			
			// 获取当前的userGrid和candidates数据
			const currentUserGrid = get(userGrid);
			const currentCandidates = get(candidates);
			
			// 1. 清除当前所有候选数字
			for (const key in currentCandidates) {
				const [x, y] = key.split(',').map(Number);
				candidates.clear({ x, y });
			}
			
			// 2. 逐个单元格恢复网格数据
			for (let y = 0; y < savedGrid.length; y++) {
				for (let x = 0; x < savedGrid[y].length; x++) {
					userGrid.set({ x, y }, savedGrid[y][x]);
				}
			}
			
			// 3. 恢复候选数字数据
			for (const key in savedCandidates) {
				const [x, y] = key.split(',').map(Number);
				const candidateList = savedCandidates[key];
				
				// 添加每个候选数字
				for (const num of candidateList) {
					if (num !== undefined) { // 过滤掉可能的undefined值
						candidates.add({ x, y }, num);
					}
				}
			}
			
			// 4. 恢复笔记模式状态
			const currentNotes = get(notes);
			if (currentNotes !== memorySave.notes) {
				notes.toggle();
			}
			
			console.log('已成功恢复Grid数据到棋盘');
			console.log('恢复的网格数据:', savedGrid);
		} else {
			console.log('没有可恢复的保存数据');
		}
  	}
</script>

<div class="action-buttons space-x-3">

	<button class="btn btn-round" disabled={$gamePaused} title="Undo" on:click={saveSnapshot}>保存</button>

	<button class="btn btn-round" disabled={$gamePaused} title="Redo" on:click={loadSnapshot}>恢复</button>

	<button class="btn btn-round" disabled={$gamePaused} title="Undo">
		<svg class="icon-outline" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor">
			<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M3 10h10a8 8 0 018 8v2M3 10l6 6m-6-6l6-6" />
		</svg>
	</button>

	<button class="btn btn-round" disabled={$gamePaused} title="Redo">
		<svg class="icon-outline" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor">
			<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M21 10h-10a8 8 90 00-8 8v2M21 10l-6 6m6-6l-6-6" />
		</svg>
	</button>

	<button class="btn btn-round btn-badge" disabled={$keyboardDisabled || !hintsAvailable || $userGrid[$cursor.y][$cursor.x] !== 0} on:click={handleHint} title="Hints ({$hints})">
		<svg class="icon-outline" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor">
			<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M9.663 17h4.673M12 3v1m6.364 1.636l-.707.707M21 12h-1M4 12H3m3.343-5.657l-.707-.707m2.828 9.9a5 5 0 117.072 0l-.548.547A3.374 3.374 0 0014 18.469V19a2 2 0 11-4 0v-.531c0-.895-.356-1.754-.988-2.386l-.548-.547z" />
		</svg>

		{#if $settings.hintsLimited}
			<span class="badge" class:badge-primary={hintsAvailable}>{$hints}</span>
		{/if}
	</button>

	<button class="btn btn-round btn-badge" on:click={notes.toggle} title="Notes ({$notes ? 'ON' : 'OFF'})">
		<svg class="icon-outline" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24" stroke="currentColor">
			<path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M15.232 5.232l3.536 3.536m-2.036-5.036a2.5 2.5 0 113.536 3.536L6.5 21.036H3v-3.572L16.732 3.732z" />
		</svg>

		<span class="badge tracking-tighter" class:badge-primary={$notes}>{$notes ? 'ON' : 'OFF'}</span>
	</button>

</div>


<style>
	.action-buttons {
		@apply flex flex-wrap justify-evenly self-end;
	}

	.btn-badge {
		@apply relative;
	}

	.badge {
		min-height: 20px;
		min-width:  20px;
		@apply p-1 rounded-full leading-none text-center text-xs text-white bg-gray-600 inline-block absolute top-0 left-0;
	}

	.badge-primary {
		@apply bg-primary;
	}
</style>
