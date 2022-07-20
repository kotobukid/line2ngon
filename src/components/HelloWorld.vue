<template lang="pug">
    svg(width="400" height="320" viewBox="0 0 400 320")
        rect(x="0" y="0" width="400" height="320" fill="black")
        line(:x1="line.x1" :y1="line.y1" :x2="line.x2" :y2="line.y2" :stroke-width="line.width" stroke="brown" fill="none")
        path(:d="polygon_points_text" fill="none" stroke="white" stroke-width="1")
        circle.p.p1(:cx="polygon.points[0].x" :cy="polygon.points[0].y" r="3" fill="red")
        circle.p.p2(:cx="polygon.points[1].x" :cy="polygon.points[1].y" r="3" fill="blue")
        circle.p.p3(:cx="polygon.points[2].x" :cy="polygon.points[2].y" r="3" fill="green")
        circle.p.p4(:cx="polygon.points[3].x" :cy="polygon.points[3].y" r="3" fill="purple")
        rect(x="0" y="0" fill="transparent" stroke-width="0" stroke="transparent" width="400" height="320" @pointerdown="pd")

</template>

<script lang="ts">
import {Component, Vue} from 'vue-property-decorator';
import _ from 'lodash';

declare type LineSource = {
    x1: number,
    y1: number,
    x2: number,
    y2: number,
    width: number
}
declare type Point2D = {
    x: number,
    y: number
}
declare type nGon = {
    points: Point2D[]
}

const line2nGon = (line: LineSource): nGon => {
    const distance: number = line.width / 2;
    const rad90: number = Math.PI / 2;
    const theta: number = Math.atan2(line.y2 - line.y1, line.x2 - line.x1);

    const p0: Point2D = {
        x: line.x2 + Math.cos(theta + rad90) * distance,
        y: line.y2 + Math.sin(theta + rad90) * distance
    };
    const p1: Point2D = {
        x: line.x1 + Math.cos(theta + rad90) * distance,
        y: line.y1 + Math.sin(theta + rad90) * distance
    };
    const p2: Point2D = {
        x: line.x1 + Math.cos(theta - rad90) * distance,
        y: line.y1 + Math.sin(theta - rad90) * distance
    };
    const p3: Point2D = {
        x: line.x2 + Math.cos(theta - rad90) * distance,
        y: line.y2 + Math.sin(theta - rad90) * distance
    };

    return {
        points: [
            p0,
            p1,
            p2,
            p3,
        ]
    };
}

@Component
export default class HelloWorld extends Vue {
    line: LineSource = {
        x1: 10,
        y1: 10,
        x2: 300,
        y2: 275,
        width: 8.5 * 2
    }

    get polygon(): nGon {
        return line2nGon(this.line);
    }

    get polygon_points_text(): string {
        const points: Point2D[] = this.polygon.points;
        return _.map<Point2D, string>(points, (p: Point2D, index: number): string => {
            if (index === 0) {
                return `M ${p.x},${p.y}`;
            } else if (index === points.length - 1) {
                return `L ${p.x},${p.y} Z`;
            } else {
                return `L ${p.x},${p.y}`;
            }
        }).join(' ');
    }

    pd(e: PointerEvent): void {
        this.line = {...this.line, x2: e.offsetX, y2: e.offsetY};
    }
}
</script>

<style scoped lang="less">
svg {
    outline: 1px solid grey;
}

circle.p {
    stroke: white;
    stroke-width: 1px;
}
</style>