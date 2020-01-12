# Arm Computation Library


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
1. executor_stream
2. layers
3. types


## graphs
1. node
2. edge
3. graph
4. workload

