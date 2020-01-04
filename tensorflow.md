

## op registry

```cpp
// what to register
struct OpRegistrationData {
    OpDef op_def;
    OpShapeInferenceFn shape_inference_fn;
    bool is_function_op = false;
};


// where to store
table map
mutable std::unordered_map<string, const OpRegistrationData*> registry_

// how to register to table map
Status OpRegistry::RegisterAlreadyLocked(
        const OpRegistrationDataFactory& op_data_factory){}


// builder
class OpDefBuilder{
    public:
        OpDefBuilder& Attr(string spec);
        OpDefBuilder& Input(string spec);
        ...
            OpDefBuilder& SetShapeFn(OpShapeInferenceFn fn);
        Status Finalize(OpRegistrationData* op_reg_data) const;
    private:
        OpRegistrationData op_reg_data_;
        std::vector<string> attrs_;
        std::vector<string> inputs_;
        std::vector<string> outputs_;
        std::vector<string> control_outputs_;
        string doc_;
        std::vector<string> errors_;
};


// wrapper
// used for selective registration(register or not)
// true
template<>
class OpDefBuilderWrapper<true>{
    explicit OpDefBuilderWrapper(const char name[]) : builder_(name) {}
    OpDefBuilderWrapper<true>& Attr(string spec) {
        builder_.Attr(std::move(spec));
        return *this;
    }
    OpDefBuilderWrapper<true>& Input(string spec) {
        builder_.Input(std::move(spec));
        return *this;
    }
};
// false(no op, just return directly)
template <>
class OpDefBuilderWrapper<false> {
    public:
        explicit constexpr OpDefBuilderWrapper(const char name[]) {}
        OpDefBuilderWrapper<false>& Attr(StringPiece spec) { return *this; }
};


// receiver
// its' constructor is used to register
struct OpDefBuilderReceiver {
    // To call OpRegistry::Global()->Register(...), used by the
    // REGISTER_OP macro below.
    // Note: These are implicitly converting constructors.
    OpDefBuilderReceiver(
            const OpDefBuilderWrapper<true>& wrapper);  // NOLINT(runtime/explicit)
    constexpr OpDefBuilderReceiver(const OpDefBuilderWrapper<false>&) {
    }  // NOLINT(runtime/explicit)
};
OpDefBuilderReceiver::OpDefBuilderReceiver(
        const OpDefBuilderWrapper<true>& wrapper) {
    OpRegistry::Global()->Register(
            [wrapper](OpRegistrationData* op_reg_data) -> Status {
            return wrapper.builder().Finalize(op_reg_data);
            });
}




// macro
#define REGISTER_OP(name) REGISTER_OP_UNIQ_HELPER(__COUNTER__, name)
#define REGISTER_OP_UNIQ_HELPER(ctr, name) REGISTER_OP_UNIQ(ctr, name)
#define REGISTER_OP_UNIQ(ctr, name)                                          \
    static ::tensorflow::register_op::OpDefBuilderReceiver register_op##ctr    \
    TF_ATTRIBUTE_UNUSED =                                                  \
    ::tensorflow::register_op::OpDefBuilderWrapper<SHOULD_REGISTER_OP( \
            name)>(name)
```


