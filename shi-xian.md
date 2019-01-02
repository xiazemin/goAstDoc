```
package main

import "fmt"

type NodeValueType uint8

const (
    TypeUnknown = NodeValueType(iota)
    TypeInteger
    TypeFloat
    TypeString
    TypeBool
)

type NodeValue struct {
    Typ   NodeValueType
    Value interface{}
}

type Environment struct {
    envName string
    store   map[string]NodeValue
}

func NewEnvironment(name string) *Environment {
    return &Environment{
        envName: name,
        store:   make(map[string]NodeValue),
    }
}

func (e *Environment) SaveVariable(varName string, value NodeValue) {
    e.store[varName] = value
}

func (e *Environment) GetVariable(varName string) NodeValue {
    value, exists := e.store[varName]
    if !exists {
        panic("variable not found")
    }
    return value
}

func (r *NodeValue) ToString() string {
    switch r.Typ {
    case TypeInteger, TypeString, TypeFloat, TypeBool:
        return fmt.Sprint(r.Value)
    default:
        return ""
    }
}

func (r *NodeValue) ToFloat() float64 {
    switch r.Typ {
    case TypeFloat:
        return r.Value.(float64)
    case TypeInteger:
        return float64(r.Value.(int64))
    default:
        panic("Unexcept Behavior.")
    }
}

type Node interface {
    Eval() NodeValue
}

//整数节点
type IntegerNode struct {
    val int64
}

func NewIntegerNode(v int64) *IntegerNode {
    return &IntegerNode{val: v}
}

func (n *IntegerNode) Eval() NodeValue {
    return NodeValue{
        Typ:   TypeInteger,
        Value: n.val,
    }
}

//float节点
type FloatNode struct {
    val float64
}

func NewFloatNode(v float64) *FloatNode {
    return &FloatNode{val: v}
}

func (n *FloatNode) Eval() NodeValue {
    return NodeValue{
        Typ:   TypeFloat,
        Value: n.val,
    }
}

//string节点
type StringNode struct {
    val string
}

func NewStringNode(v string) *StringNode {
    return &StringNode{val: v}
}

func (n *StringNode) Eval() NodeValue {
    return NodeValue{
        Typ:   TypeString,
        Value: n.val,
    }
}

//bool节点
type BoolNode struct {
    val bool
}

func NewBoolNode(v bool) *BoolNode {
    return &BoolNode{val: v}
}

func (n *BoolNode) Eval() NodeValue {
    return NodeValue{
        Typ:   TypeBool,
        Value: n.val,
    }
}

//声明变量
type DeclarationNode struct {
    varName string
    val     Node
    env     *Environment
}

func NewDeclarationNode(environment *Environment, name string, n Node) *DeclarationNode {
    return &DeclarationNode{
        varName: name,
        val:     n,
        env:     environment,
    }
}

func (n *DeclarationNode) Eval() NodeValue {
    value := n.val.Eval()
    n.env.SaveVariable(n.varName, value)
    return value
}

//获取变量
type VariableNode struct {
    varName string
    env     *Environment
}

func NewVariableNode(environment *Environment, name string) *VariableNode {
    return &VariableNode{
        varName: name,
        env:     environment,
    }
}

func (n *VariableNode) Eval() NodeValue {
    return n.env.GetVariable(n.varName)
}

//加法节点
type AddNode struct {
    left  Node
    right Node
}

func NewAddNode(l, r Node) *AddNode {
    return &AddNode{
        left:  l,
        right: r,
    }
}

func (n *AddNode) Eval() NodeValue {
    leftEval := n.left.Eval()
    rightEval := n.right.Eval()

    switch {
    case leftEval.Typ == TypeString || rightEval.Typ == TypeString:
        return NodeValue{
            Typ:   TypeString,
            Value: leftEval.ToString() + rightEval.ToString(),
        }
    case leftEval.Typ == TypeFloat || rightEval.Typ == TypeFloat:
        return NodeValue{
            Typ:   TypeString,
            Value: leftEval.ToFloat() + rightEval.ToFloat(),
        }
    case leftEval.Typ == TypeInteger && rightEval.Typ == TypeInteger:
        return NodeValue{
            Typ:   TypeInteger,
            Value: leftEval.Value.(int64) + rightEval.Value.(int64),
        }
    default:
        return NodeValue{
            Typ:   TypeUnknown,
            Value: nil,
        }
    }
}

func main() {
    globalEnv := NewEnvironment("global")

    //var a = "this is a test[" + 123 + "]"
    declareAst := NewDeclarationNode(globalEnv, "a",
        NewAddNode(
            NewAddNode(
                NewStringNode("this is a test["),
                NewIntegerNode(123)),
            NewStringNode("]")))
    declareAst.Eval()

    // a + 456
    ast := NewAddNode(
        NewVariableNode(globalEnv, "a"),
        NewIntegerNode(456))
    fmt.Println("Start Eval:")
    r := ast.Eval()
    if r.Typ == TypeString {
        fmt.Println(r.Value.(string))
    }
}
```



