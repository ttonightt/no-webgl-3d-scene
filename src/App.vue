<script setup>
import { onMounted, ref, useTemplateRef, watch } from "vue";
const pi2 = Math.PI * 2;


const vec = (x, y, z) => ({ x, y, z });

const cross = (v1, v2) => ({
	x: (v1.y * v2.z) - (v1.z * v2.y),
	y: (v1.z * v2.x) - (v1.x * v2.z),
	z: (v1.x * v2.y) - (v1.y * v2.x)
});

const dot = (v1, v2) => (v1.x * v2.x) + (v1.y * v2.y) + (v1.z * v2.z);

const norm = v => {

	const mag = Math.sqrt( (v.x ** 2) + (v.y ** 2) + (v.z ** 2) );

	return {
		x: v.x / mag,
		y: v.y / mag,
		z: v.z / mag
	};
};

const diff = (v, v0) => ({
	x: v.x - v0.x,
	y: v.y - v0.y,
	z: v.z - v0.z
});

const lookAtM = (targetV, cameraV) => {

	const UP = vec(0, 1, 0);

	const L = norm(diff(targetV, cameraV));
	const s = norm(cross(L, UP));
	const u = cross(s, L);

	return [
		s.x, s.y, s.z, -dot(s, cameraV),
		u.x, u.y, u.z, -dot(u, cameraV),
		-L.x, -L.y, -L.z, dot(L, cameraV),
		0, 0, 0, 1
	];
};

const intrinsicM = (f, plainW, plainH) => [

	f, 0, plainW / 2,
	0, f, plainH / 2,
	0, 0, 1
];

const proj = (x0, y0, z0) => {

	const trg = vec(0, 0, 0);
	const cam = vec(0, 100, 100);
	const f = 100;

	const E = lookAtM(trg, cam);
	const I = intrinsicM(f, window.innerWidth, window.innerHeight);

	const V = [
		(E[0] * x0) + (E[1] * y0) +  (E[2] * z0) + E[3],
		(E[4] * x0) + (E[5] * y0) +  (E[6] * z0) + E[7],
		(E[8] * x0) + (E[9] * y0) + (E[10] * z0) + E[11]
	];

	const P = [
		(I[0] * V[0]) + (I[1] * V[1]) + (I[2] * V[2]),
		(I[3] * V[0]) + (I[4] * V[1]) + (I[5] * V[2]),
		(I[6] * V[0]) + (I[7] * V[1]) + (I[8] * V[2]),
	];

	return [ P[0] / P[2], P[1] / P[2] ];
};

class Spine {

	constructor (R, r, n) {

		this.R = R;
		this.r = r;
		this.n = n;
		this.o = [0, 0, 0];
	}

	moveTo (v) {
		this.o = [...v];
	}

	fx (t) {
		return (this.R + this.r * Math.cos(this.n * t)) * Math.cos(t);
	}

	fy (t) {
		return (this.R + this.r * Math.cos(this.n * t)) * Math.sin(t);
	}

	fz (t) {
		return this.r * Math.sin(this.n * t);
	}
}


const canvasRef = useTemplateRef("canvas-ref");

onMounted(() => {

	const cnv = canvasRef.value;

	cnv.width = window.innerWidth;
	cnv.height = window.innerHeight;

	const ctx = canvasRef.value.getContext("2d");

	ctx.strokeStyle = "#ffffff";
	ctx.fillStyle = "#ffffff";
	ctx.lineCap = "round";
	ctx.lineWidth = 1;

	const Ox = 0, Oy = 0;

	const spine = new Spine(100, 10, 14);

	let t0 = 0;

	const render = () => {

		ctx.clearRect(0, 0, cnv.width, cnv.height);

		//const gap = Math.PI / 5 + t0;
		const gap = 0;
		let t = -t0;
		const dt = 0.1 / spine.n;
	
		let point = proj( spine.fx(t), spine.fy(t), spine.fz(t) );

		for (t; t < pi2 + dt - gap; t += dt) {

			//ctx.lineWidth = 8 * Math.abs( (2 * t / pi2) - 1 ) + 8;

			ctx.beginPath();
			ctx.moveTo(Ox + point[0], Oy + point[1]);

			point = proj( spine.fx(t), spine.fy(t), spine.fz(t) );
			ctx.lineTo(Ox + point[0], Oy + point[1]);

			ctx.stroke();
		}

		//ctx.beginPath();
		//ctx.moveTo(...center( proj( spark[0], spark[1], spark[2] ) ) );

		//for (let i = 3; i < spark.length - 8; i += 9) {

		//	ctx.bezierCurveTo(
		//		...center( proj( spark[i], spark[i + 1], spark[i + 2] ) ),
		//		...center( proj( spark[i + 3], spark[i + 4], spark[i + 5] ) ),
		//		...center( proj( spark[i + 6], spark[i + 7], spark[i + 8] ) )
		//	);
		//}

		//ctx.lineWidth = 1;
		//ctx.fill();
		//ctx.stroke();

		//t0 = (t0 + 0.01) % pi2;

		//requestAnimationFrame(anim);
	};

	render();

});
</script>

<template>
	<canvas ref="canvas-ref"></canvas>
</template>

<style scoped>
canvas {
	background-color: black;
	display: block;
}
</style>
