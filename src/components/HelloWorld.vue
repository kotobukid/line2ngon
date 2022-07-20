<template lang="pug">
    svg(width="400" height="320" viewBox="0 0 400 320")
        rect(x="0" y="0" width="400" height="320" fill="black")
        g.line
            line(:x1="line.x1" :y1="line.y1" :x2="line.x2" :y2="line.y2" :stroke-width="line.width" stroke="brown" fill="none")
        path(:d="polygon_points_text" fill="none" stroke="white" stroke-width="1")
        circle.p.p1(:cx="polygon.points[0].x" :cy="polygon.points[0].y" r="3" fill="red")
        circle.p.p2(:cx="polygon.points[1].x" :cy="polygon.points[1].y" r="3" fill="blue")
        circle.p.p3(:cx="polygon.points[2].x" :cy="polygon.points[2].y" r="3" fill="green")
        circle.p.p4(:cx="polygon.points[3].x" :cy="polygon.points[3].y" r="3" fill="purple")
        g.marks
            circle.mark(v-for="m in marks" :cx="m.x" :cy="m.y" r="2" :fill="m.included2 ? 'red' : m.included1 ? 'pink': 'grey'")
        g.line
            circle(fill="lightgreen" :cx="line.x1" :cy="line.y1" r="2")
            circle(fill="lightgreen" :cx="line.x2" :cy="line.y2" r="2")
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

declare type Mark = {
    included1: boolean,
    included2: boolean
} & Point2D

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

export declare type LongLineA = { points: [Point2D, Point2D] };
declare type LineSimple = [Point2D, Point2D];
const point_to_long_line = (p: Point2D): LongLineA => {
    return {points: [p, {x: p.x + 1000000, y: p.y}]};
};
const cross_check = (a: Point2D, b: Point2D, c: Point2D, d: Point2D): 1 | 0 => {
    // https://qiita.com/zu_rin/items/e04fdec4e3dec6072104
    let s = (a.x - b.x) * (c.y - a.y) - (a.y - b.y) * (c.x - a.x);
    let t = (a.x - b.x) * (d.y - a.y) - (a.y - b.y) * (d.x - a.x);
    if (s * t > 0) {
        return 0;
    }

    s = (c.x - d.x) * (a.y - c.y) - (c.y - d.y) * (a.x - c.x);
    t = (c.x - d.x) * (b.y - c.y) - (c.y - d.y) * (b.x - c.x);
    if (s * t > 0) {
        return 0;
    }
    return 1;
};

const getNGonBBox = (points: Point2D[]): { x1: number, y1: number, x2: number, y2: number } => {
    let x1 = points[0].x;
    let y1 = points[0].y;
    let x2 = points[0].x;
    let y2 = points[0].y;

    for (let i = 1; i < points.length; i++) {
        x1 = Math.min(x1, points[i].x);
        y1 = Math.min(y1, points[i].y);
        x2 = Math.max(x2, points[i].x);
        y2 = Math.max(y2, points[i].y);
    }

    return {
        x1, y1, x2, y2
    }
}

const check_point_in_ngon_points = (points: Point2D[], point: Point2D): boolean => {
    const long_line = point_to_long_line(point);
    let crossing_count: number = 0;
    for (let i = 0; i < points.length; i++) {
        const line: LineSimple = [points[i], points[i === points.length - 1 ? 0 : i + 1]];
        crossing_count += cross_check(long_line.points[0], long_line.points[1], line[0], line[1]);
    }

    return crossing_count % 2 === 1;
};


@Component
export default class HelloWorld extends Vue {
    line: LineSource = {
        x1: 200,
        y1: 160,
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
        const line: LineSource = this.line;
        this.$nextTick(() => {
            // const marks_next: Mark[] = [];
            const marks: Mark[] = this.marks;


            const {x1, y1, x2, y2} = getNGonBBox(this.polygon.points);

            _.each(marks, (m: Mark) => {
                if (x1 < m.x && m.x < x2 && y1 < m.y && m.y < y2) {
                    m.included1 = true;
                    m.included2 = check_point_in_ngon_points(this.polygon.points, m);
                } else {
                    m.included1 = false;
                    m.included2 = false;
                }
            })

            this.marks = marks;
        });
    }

    marks: Mark[] = (() => {
        const marks: Mark[] = [];
        _.each(_.range(33), (y: number) => {
            _.each(_.range(41), (x: number) => {
                marks.push({included1: false, included2: false, x: x * 10, y: y * 10});
            });
        });
        return marks;
    })()
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

g.marks {
    pointer-events: none;
}
</style>