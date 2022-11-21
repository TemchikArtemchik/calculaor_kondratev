<template>
    <Page>
        <ActionBar title="Calculator" />
        <GridLayout columns="*, *, *, *" rows="2*, *, *, *, *, *">
            <StackLayout orientation="vertical" row="0" colSpan="4"
                verticalAlignment="bottom" horizontalAlignment="right">
                <Label class="history" :text="history" horizontalAlignment="right">
                </Label>
                <Label :text="current" horizontalAlignment="right">
                </Label>
            </StackLayout>
            <Button @tap="clear" row="1" class="controls" col="0" text="C">
            </Button>
            <Button @tap="divide" row="1" class="controls" col="1" text="/">
            </Button>
            <Button @tap="multiply" row="1" class="controls" col="2" text="*">
            </Button>
            <Button @tap="del" row="1" class="controls" col="3" text="↶">
            </Button>
            <Button @tap="addNumber('7')" row="2" class="numbers" col="0"
                text="7"></Button>
            <Button @tap="addNumber('8')" row="2" class="numbers" col="1"
                text="8"></Button>
            <Button @tap="addNumber('9')" row="2" class="numbers" col="2"
                text="9"></Button>
            <Button @tap="minus" row="2" class="controls" col="3" text="-">
            </Button>
            <Button @tap="addNumber('4')" row="3" class="numbers" col="0"
                text="4"></Button>
            <Button @tap="addNumber('5')" row="3" class="numbers" col="1"
                text="5"></Button>
            <Button @tap="addNumber('6')" row="3" class="numbers" col="2"
                text="6"></Button>
            <Button @tap="add" row="3" class="controls" col="3" text="+">
            </Button>
            <Button @tap="addNumber('1')" row="4" class="numbers" col="0"
                text="1"></Button>
            <Button @tap="addNumber('2')" row="4" class="numbers" col="1"
                text="2"></Button>
            <Button @tap="addNumber('3')" row="4" class="numbers" col="2"
                text="3"></Button>
            <Button @tap="pow" row="4" class="controls" col="3" text="xⁿ">
            </Button>
            <Button @tap="precent" row="5" class="controls" col="0" text="%">
            </Button>
            <Button @tap="addNumber('0')" row="5" class="numbers" col="1"
                text="0"></Button>
            <Button @tap="dot" row="5" class="controls" col="2" text=".">
            </Button>
            <Button @tap="equal" row="5" class="controls" col="3" text="=">
            </Button>
        </GridLayout>

    </Page>
</template>

<script>
    export default {
        data() {
            return {
                previous: [],
                current: "0",
                history: "",
                operator: null,
                operator_stack: [],
                operatorClicked: false,
                op_priority: 0,
                op_priority_stack: []
            };
        },
        methods: {
            clear() {
                this.previous = [];
                this.current = "0";
                this.history = "";
                this.operator_stack = [];
                this.operatorClicked = false;
                this.op_priority = 0;
                this.op_priority_stack = [];
            },
            del() {
                if (!this.operatorClicked) {
                    if (this.current.length == 1) {
                        this.current = "0";
                    } else {
                        this.current = String(this.current).slice(0, -1);
                    }
                }
            },
            precent() {
                if (!this.operatorClicked)
                    this.current = parseFloat(this.current) / 100;
            },
            addNumber(number) {
                if (this.operatorClicked) {
                    this.current = "";
                    this.operatorClicked = false;
                }
                if (this.current == "0") {
                    this.current = number;
                } else this.current += number;
            },
            dot() {
                if (
                    !this.operatorClicked &&
                    String(this.current).indexOf(".") == -1
                ) {
                    this.current += ".";
                }
            },
            setPrevious() {
                this.operator_stack.push(this.operator);
                this.previous.push(this.current);
                this.operatorClicked = true;
                this.op_priority_stack.push(this.op_priority);
            },
            divide() {
                if (!this.operatorClicked) {
                    this.history += this.current + "/";
                    if (this.previous != [] && this.op_priority >= 1) {
                        this.op_priority = 1;
                        this.result();
                    }
                    this.operator = (a, b) => a / b;
                    this.op_priority = 1;
                    this.setPrevious();
                }
            },
            multiply() {
                if (!this.operatorClicked) {
                    this.history += this.current + "*";
                    if (this.previous != [] && this.op_priority >= 1) {
                        this.op_priority = 1;
                        this.result();
                    }
                    this.operator = (a, b) => a * b;
                    this.op_priority = 1;
                    this.setPrevious();
                }
            },
            add() {
                if (!this.operatorClicked) {
                    this.history += this.current + "+";
                    if (this.previous != [] && this.op_priority >= 0) {
                        this.op_priority = 0;
                        this.result();
                    }
                    this.operator = (a, b) => a + b;
                    this.op_priority = 0;
                    this.setPrevious();
                }
            },
            minus() {
                if (!this.operatorClicked) {
                    this.history += this.current + "-";
                    if (this.previous != [] && this.op_priority >= 0) {
                        this.op_priority = 0;
                        this.result();
                    }
                    this.operator = (a, b) => a - b;
                    this.op_priority = 0;
                    this.setPrevious();
                }
            },
            pow() {
                if (!this.operatorClicked) {
                    this.history += this.current + "^";
                    if (this.previous != [] && this.op_priority >= 2) {
                        this.op_priority = 2;
                        this.result();
                    }
                    this.operator = (a, b) => Math.pow(a, b);
                    this.op_priority = 2;
                    this.setPrevious();
                }
            },
            result() {
                while (
                    this.operator_stack.length != 0 &&
                    this.op_priority_stack.length != 0 &&
                    this.op_priority <=
                    this.op_priority_stack[this.op_priority_stack.length - 1]
                ) {
                    this.op_priority_stack.pop();
                    this.operator = this.operator_stack.pop();
                    this.current = this.operator(
                        parseFloat(this.previous.pop()),
                        parseFloat(this.current)
                    );
                    if (this.current == Infinity) {
                        alert({
                            title: "Ошибка!",
                            message: "Попытка деления на ноль или переполнение.",
                            okButtonText: "ОК"
                        }).then(() => {
                            this.clear();
                        });
                    }
                    if (isNaN(this.current)) {
                        alert({
                            title: "Ошибка!",
                            message: "Результат не определен.",
                            okButtonText: "ОК"
                        }).then(() => {
                            this.clear();
                        });
                    }
                }
            },
            equal() {
                this.history = "";
                this.op_priority = 0;
                this.result();
                this.operatorClicked = false;
                this.previous = [];
                this.operator_stack = [];
                this.op_priority_stack = [];
            }
        }
    };
</script>

<style scoped>
    ActionBar {
        background-color: black;
        color: white;
    }

    GridLayout {
        background-color: black;
    }

    StackLayout {
        height: 100%;
    }

    Label {
        font-size: 42;
        padding-right: 15px;
        padding-bottom:  20px;
        color: aquamarine;
    }

    .history {
        font-size: 24;
    }

    button {
        font-size: 36;
        width: 100%;
        height: 100%;
    }

    .controls {
        background-color: red;
        color: white;
    }

    .numbers {
        background-color: red;
        color: rgb(255, 255, 255);
    }
</style>