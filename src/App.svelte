<script>
	import { onMount } from 'svelte';
	import { validateSencode } from '@sudoku/sencode';
	import game from '@sudoku/game';
	import { modal } from '@sudoku/stores/modal';
	import { gameWon } from '@sudoku/stores/game';
	import { guideActions } from '@sudoku/stores/guide';
	import Board from './components/Board/index.svelte';
	import Controls from './components/Controls/index.svelte';
	import Header from './components/Header/index.svelte';
	import Modal from './components/Modal/index.svelte';
	import GuideOverlay from './components/Overlay/GuideOverlay.svelte';

	gameWon.subscribe(won => {
		if (won) {
			game.pause();
			modal.show('gameover');
		}
	});

	onMount(() => {
		let hash = location.hash;

		if (hash.startsWith('#')) {
			hash = hash.slice(1);
		}

		let sencode;
		if (validateSencode(hash)) {
			sencode = hash;
		}

		// modal.show('welcome', { onHide: game.resume, sencode });
		modal.show('welcome', { 
			onHide: () => {
				// welcome关闭后执行
				// 检查是否第一次访问
				const hasSeen = sessionStorage.getItem('sudokuGuideSeen');
				if (!hasSeen) {
					setTimeout(() => {
						guideActions.start();
						sessionStorage.setItem('sudokuGuideSeen', 'true');
					}, 300);
				}
				else {
					game.resume();// 不是第一次访问，直接恢复游戏
				}
				
			}, 
			sencode 
		});



	});
</script>

<!-- Timer, Menu, etc. -->
<header>
	<Header />
</header>

<!-- Sudoku Field -->
<section>
	<Board />
</section>

<!-- Keyboard -->
<footer>
	<Controls />
</footer>

<Modal />

<GuideOverlay />

<style global>
	@import "./styles/global.css";
</style>