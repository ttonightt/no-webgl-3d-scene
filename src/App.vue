<script setup>
import { onMounted, useTemplateRef } from "vue";

const canvasRef = useTemplateRef("canvas-ref");

const R = 100;
const n = 20;
const r = 20;
const pi2 = Math.PI * 2;
const Ox = 300, Oy = 300;

const fx = t => (R + r * Math.cos(n * t)) * Math.cos(t);
const fy = t => (R + r * Math.cos(n * t)) * Math.sin(t);
const fz = t => r * Math.sin(n * t);

onMounted(() => {

	const cnv = canvasRef.value;

	cnv.width = window.innerWidth;
	cnv.height = window.innerHeight;

	const ctx = canvasRef.value.getContext("2d");

	let t = 0;
	const dt = 0.25 / n;

	ctx.lineWidth = 1;
	ctx.strokeStyle = "#000";

	ctx.beginPath();
	ctx.moveTo(Ox + fy(t), Oy + fz(t));

	for (t; t < pi2 + dt; t += dt) {

		ctx.lineTo(Ox + fy(t), Oy + fz(t));
	}

	ctx.closePath();
	ctx.stroke();
});
</script>

<template>
	<canvas ref="canvas-ref"></canvas>
</template>

<style scoped>
canvas {
	background-color: aliceblue;
	display: block;
}
</style>
