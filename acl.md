# Arm Computation Library

## schduler and context
```cpp
class Scheduler
{
public:
/** Scheduler type */
enum class Type
{
ST,    /**< Single thread. */
CPP,   /**< C++11 threads. */
OMP,   /**< OpenMP. */
CUSTOM /**< Provided by the user. */
};
static void set(std::shared_ptr<IScheduler> scheduler);
static IScheduler &get();
static void set(Type t);
static Type get_type();
static bool is_available(Type t);

private:
static Type                        _scheduler_type;
static std::shared_ptr<IScheduler> _custom_scheduler;
static std::map<Type, std::unique_ptr<IScheduler>> _schedulers;
// disable constructor
Scheduler();
};
```



## memory management

1. allocate device using cl::Buffer and clCreateBuffer in the bottom level


2. MemoryRegion
```cpp
class IMemoryRegion{
public:
virtual void *buffer()=0;
virtual void *buffer()const=0;
};

// CL interface
class ICLMemoryRegion : public IMemoryRegion
{
public:
const cl::Buffer &cl_data() const;
virtual void *ptr() = 0;
virtual void *map(cl::CommandQueue &q, bool blocking) = 0;
virtual void unmap(cl::CommandQueue &q) = 0;

// Inherited methods overridden :
void                          *buffer() override;
void                          *buffer() const override;
std::unique_ptr<IMemoryRegion> extract_subregion(size_t offset, size_t size) override;

protected:
cl::CommandQueue _queue;
cl::Context      _ctx;
void            *_mapping;
cl::Buffer       _mem;
};
```


3. allocator
```cpp
void *Allocator::allocate(size_t size, size_t alignment)
{
ARM_COMPUTE_UNUSED(alignment);
return ::operator new(size);
}

void Allocator::free(void *ptr)
{
::operator delete(ptr);
}

std::unique_ptr<IMemoryRegion> Allocator::make_region(size_t size, size_t alignment)
{
return arm_compute::support::cpp14::make_unique<MemoryRegion>(size, alignment);
}

CLBufferAllocator::CLBufferAllocator(CLCoreRuntimeContext *ctx)
: _ctx(ctx)
{
}

void *CLBufferAllocator::allocate(size_t size, size_t alignment)
{
ARM_COMPUTE_UNUSED(alignment);
cl_mem buf;
if(_ctx == nullptr)
{
buf = clCreateBuffer(CLScheduler::get().context().get(), CL_MEM_ALLOC_HOST_PTR | CL_MEM_READ_WRITE, size, nullptr, nullptr);
}
else
{
buf = clCreateBuffer(_ctx->context().get(), CL_MEM_ALLOC_HOST_PTR | CL_MEM_READ_WRITE, size, nullptr, nullptr);
}
return static_cast<void *>(buf);
}

void CLBufferAllocator::free(void *ptr)
{
ARM_COMPUTE_ERROR_ON(ptr == nullptr);
clReleaseMemObject(static_cast<cl_mem>(ptr));
}

std::unique_ptr<IMemoryRegion> CLBufferAllocator::make_region(size_t size, size_t alignment)
{
ARM_COMPUTE_UNUSED(alignment);
return arm_compute::support::cpp14::make_unique<CLBufferMemoryRegion>(_ctx, CL_MEM_ALLOC_HOST_PTR | CL_MEM_READ_WRITE, size);
}
} // namespace arm_compute
```


## backend
```cpp
// tune best lws and allocate tensor or buffer
class CLDeviceBackend final : public IDeviceBackend
{
public:
void set_kernel_tuning(bool enable_tuning);
void set_kernel_tuning_mode(CLTunerMode tuning_mode);

// Inherited overridden methods
void initialize_backend() override;
void setup_backend_context(GraphContext &ctx) override;
void release_backend_context(GraphContext &ctx) override;
bool                           is_backend_supported() override;
IAllocator                    *backend_allocator() override;
std::unique_ptr<ITensorHandle> create_tensor(const Tensor &tensor) override;
std::unique_ptr<ITensorHandle> create_subtensor(ITensorHandle *parent, TensorShape shape, Coordinates coords, bool extend_parent) override;
std::unique_ptr<arm_compute::IFunction> configure_node(INode &node, GraphContext &ctx) override;
Status validate_node(INode &node) override;
std::shared_ptr<arm_compute::IMemoryManager> create_memory_manager(MemoryManagerAffinity affinity) override;
std::shared_ptr<arm_compute::IWeightsManager> create_weights_manager() override;

private:
int                                   _context_count; /**< Counts how many contexts are currently using the backend */
CLTuner                               _tuner;         /**< CL kernel tuner */
std::unique_ptr<CLBufferAllocator>    _allocator;     /**< CL buffer affinity allocator */
std::string                           _tuner_file;    /**< Filename to load/store the tuner's values from */
};
```

## frontend
```cpp
// stream interface
class IStream
{
public:
    virtual void add_layer(ILayer &layer) = 0;
    virtual const Graph &graph() const = 0;
    NodeID tail_node()
    {
        return _tail_node;
    }
    StreamHints &hints()
    {
        return _hints;
    }
    void forward_tail(NodeID nid)
    {
        _tail_node = (nid != NullTensorID) ? nid : _tail_node;
    }

protected:
    StreamHints _hints     = {};              /**< Execution and algorithmic hints */
    NodeID      _tail_node = { EmptyNodeID }; /**< NodeID pointing to the last(tail) node of the graph */
};
// substream just override Isubstream
class substream{};

// stream
class stream{
    public:
    void finalize(Target target, const GraphConfig & config);
    void run();
    private:
    GraphContext _ctx;
    GraphManager _manager;
    Graph _g;
    };

// layers
class ILayer{
    public:
    virutal NodeID create_layer(IStream &s)=0;
    ILayer &set_name(std::string name){
        _name=name;
        return *this;
        }
    }

```
2. layers
3. types


## graphs
1. node
2. edge
3. graph
4. workload


## core class

some classes about tensor
```cpp
class Tensor{
    };

class ITensorHandle
{
public:
    virtual void allocate() = 0;
    virtual void free() = 0;
    /** Set backend tensor to be managed by a memory group
     *
     * @param[in] mg Memory group
     */
    virtual void manage(IMemoryGroup *mg) = 0;
    /** Maps backend tensor object
     *
     * @param[in] blocking Flags if the mapping operations should be blocking
     */
    virtual void map(bool blocking) = 0;
    /** Un-maps a backend tensor object */
    virtual void unmap() = 0;
    /** Releases backend tensor if is marked as unused
     *
     *
     * @note This has no effect on sub-tensors
     * @warning Parent tensors don't keep track of sub-tensors,
     *          thus if a parent is set as unused then all sub-tensors will be invalidated,
     *          on the other hand if a sub-tensor is marked as unused then the parent tensor won't be released
     */
    virtual void release_if_unused() = 0;
    /** Backend tensor object accessor */
    virtual arm_compute::ITensor &tensor() = 0;
    /** Backend tensor object const accessor */
    virtual const arm_compute::ITensor &tensor() const = 0;
    /** Return the parent tensor handle if is a subtensor else this
     *
     * @return Parent tensor handle
     */
    virtual ITensorHandle *parent_handle() = 0;
    /** Checks if a backing tensor is a sub-tensor object or not
     *
     * @return True if the backend tensor is a sub-tensor else false
     */
    virtual bool is_subtensor() const = 0;
    /** Returns target type
     *
     * @return Target type
     */
    virtual Target target() const = 0;
};

class TensorInfo{
    };

class SubTensor{
    };

class TensorAllocator{
    data_layout: "NCHW" or "NHWC"
    };

class TensorShape{
    };
```


## utils
* how to run kernel
    ** add argument_2d
    ** enqueue(kernel)

```cpp
//window
//dimension
    void ICLSimple2DKernel::run(const Window &window, cl::CommandQueue &queue)
    {
    ARM_COMPUTE_ERROR_ON_UNCONFIGURED_KERNEL(this);
    ARM_COMPUTE_ERROR_ON_INVALID_SUBWINDOW(ICLKernel::window(), window);

    Window slice = window.first_slice_window_2D();

    do
    {
    unsigned int idx = 0;
    add_2D_tensor_argument(idx, _input, slice);
    add_2D_tensor_argument(idx, _output, slice);
    enqueue(queue, *this, slice, lws_hint());
    }
    while(window.slide_window_slice_2D(slice));
    }


* handle error
```


