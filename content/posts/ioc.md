+++
title = 'Rust IOC'
date = 2024-03-26T22:16:42+08:00
summary = "Rust IOC implement"
tags = ["Rust", "IOC"]
+++

## Note

In this architecture, the `Deps` will recursively take the intersection of all dependent fields Send and Sync restrictions, which means if you want to impl a function with Arc<Self> type, once a dependency in your container is `!Sync`, your Container will not pass the compile check

for example: 

``` rust
impl<Deps> A for AImpl<Deps>
where
    Deps: AsRef<AImpl> + B + Send
{
    fn aa(self: Arc<Self>){
        // your logic here
    }
}
```

Once your BImplState has any field that is `!Sync`, the whole code will not pass compile check, because the `Arc<Self>` means the `self` requires the whole `Deps` has `Send` and `Sync` trait bounds, but one of your field does not have `Sync` bound.

## Architecture

> This inspiration is created by [TD-Sky](https://github.com/TD-Sky) who's inspired by [TOETOE55](https://github.com/TOETOE55), emphatically thanks to them!

Firstly, create dependency's trait and impl.


```rust
// path/to/domain/foo_service.rs:
pub trait FooService {
    fn foo_bar(&self);
}
```

We declare a struct named FooServiceState and implement FooService
use [dep-inj](https://crates.io/crates/dep-inj) crate, and let me show you how it expands like:
```rust
// path/to/service/foo_service.rs:
use path::to::domain::FooService;

#[derive(dep_inj::DepInj)]
#[target(FooServiceImpl)]
pub struct FooServiceState {
    bar: u64
}

impl FooServiceState {
    pub fn new(bar: u64) -> Self {
        Self {
            bar
        }
    }
}

/// In its method, this struct doesn't call method from 
/// other struct, means it doesn't depend on any other struct.
/// So there is no need to derive DepInj for it indeed.
/// We can simply code like this: 
/// ```rust
/// pub struct FooServiceImpl { 
///     bar:u64,
/// }
/// impl FooService for FooServiceImpl {
///     fn foo_bar(&self) {
///         println!("{}", self.bar);
///     }
/// }
/// ```rust
/// Here I use DepInj is only for example
/// But we intend to use field in State, so we have to specify 
impl<Deps> FooService for FooServiceImpl<Deps>
where
    Deps: AsRef<FooServiceState>
{
    fn foo_bar(&self) {
        /// &self is automatically as_ref to FooServiceState, so we can use its field.
        println!("{}", self.bar);
    }
}

// DepInj macro will expand to:
#[repr(transparent)]
/// __Deps__ specify the traits that the type to deref into need to implement.
pub struct FooServiceImpl<__Deps__: ?Sized> {
    /// To pass compile
    _marker: ::core::marker::PhantomData<FooServiceState>,
    deps: __Deps__,
}
impl<__Deps__> Copy for FooServiceImpl<__Deps__> where __Deps__: Copy {}
/// Impl deref to State for Impl
impl<__Deps__: ?Sized> ::core::ops::Deref for FooServiceImpl<__Deps__>
where
    __Deps__: AsRef<FooServiceState>,
{
    type Target = FooServiceState;
    #[inline]
    fn deref(&self) -> &Self::Target {
        self.deps.as_ref()
    }
}
impl<__Deps__: ?Sized> ::core::ops::DerefMut for FooServiceImpl<__Deps__>
where
    __Deps__: AsRef<FooServiceState> + AsMut<FooServiceState>,
{
    #[inline]
    fn deref_mut(&mut self) -> &mut Self::Target {
        self.deps.as_mut()
    }
}
impl<__Deps__> From<FooServiceImpl<__Deps__>> for FooServiceState
where
    __Deps__: Into<FooServiceState>,
{
    fn from(value: FooServiceImpl<__Deps__>) -> Self {
        value.prj().into()
    }
}
impl<__Deps__> Clone for FooServiceImpl<__Deps__>
where
    __Deps__: Clone,
{
    fn clone(&self) -> Self {
        Self {
            _marker: ::core::marker::PhantomData,
            deps: self.deps.clone(),
        }
    }
}
//...
// Some factory methods
// inj method is to transfer a borrow of generic type __Deps__
// to Self(FooServiceImpl<__Deps__> here).
// pri method is to transfer a borrow of self to a borrow of generic type __Deps__.
impl<__Deps__: ?Sized> FooServiceImpl<__Deps__> {
    #[inline]
    pub fn inj_ref(deps: &__Deps__) -> &Self {
        unsafe { &*(deps as *const __Deps__ as *const Self) }
    }
    #[inline]
    pub fn prj_ref(&self) -> &__Deps__ {
        unsafe { &*(self as *const Self as *const __Deps__) }
    }
    #[inline]
    pub fn inj_ref_mut(deps: &mut __Deps__) -> &mut Self {
        unsafe { &mut *(deps as *mut __Deps__ as *mut Self) }
    }
    #[inline]
    pub fn prj_ref_mut(&mut self) -> &mut __Deps__ {
        unsafe { &mut *(self as *mut Self as *mut __Deps__) }
    }
    //...
}
impl<__Deps__> FooServiceImpl<__Deps__> {
    #[inline]
    pub fn inj(deps: __Deps__) -> Self {
        Self {
            _marker: ::core::marker::PhantomData,
            deps,
        }
    }
    #[inline]
    pub fn prj(self) -> __Deps__ {
        self.deps
    }
}
```

Then we write container which holds all States.

And the boilerplate is to implement all Traits for the container using macro-generated type's `inj` method.

The `derive_more::AsRef` is used for add the `AsRef<FooService>` after call `inj_ref` on `&Container`, so it can call implements on `Dep` who has generic bound to `AsRef<FooService>`.

```rust
// path/to/container.rs
// The Service need to use Other dependencies only need to depend on Container
#[derive(derive_more::AsRef)]
pub struct Container {
    #[as_ref]
    pub foo_service: FooServiceState,
}

impl Container {
    // Initialize method for Container.
    pub fn new(config: &AppConfig) -> Self {
        let foo_service = FooServiceState::new(config.num);
        Self {
            foo_service
        }
    }
}

// .../boilerplate.rs
use path::to::Container;
use path::to::domain::FooService;
use path::to::foo_service::FooServiceImpl;
use path::to::bar_service::BarServiceImpl;

// Boilerplate for provide FooService implement for Container
impl FooService for Container {
    fn foo_bar(&self) {
        // Use inj_ref() generated by dep-inj macro,
        // to get &FooServiceImpl from Container
        FooServiceImpl::inj_ref(self).foo_bar()
    }
}

impl BarService for Container {
    fn bar_foo(&self) {
        BarServiceImpl::inj_ref(self).bar_foo()
    }
}
```

Then we write `BarService` who depends on generic type with `FooService` Trait.
The `dep_inj_target` macro is wrote to support `dep_inj` on no field struct.

```rust
// path/to/bar_service.rs
// Use of the Dependency Injection System!
use path::to::domain::FooService;

#[dep_inj_target::dep_inj_target]
pub struct BarServiceImpl;

impl<Deps> BarServiceImpl<Deps>
where
    Deps: FooService
{
    pub fn bar_foo(&self) {
        // 2. Use deref mechanism to auto use method under trait `FooService`
        self.prj_ref().foo_bar()
        // Or `(self.prj_ref() as &dyn FooService).foo_bar()`
    }
}

```

Finally we can use Entry to call `bar_foo` from `BarServiceImpl` and in it, call `foo_bar` from `FooServiceImpl` successfully!

```rust
// .../entry.rs
use path::to::Container;

pub struct Entry {
    container: Container
}

impl Entry {
    fn run_bar_foo(&self) {
        // The container implements the `FooService` bound, so it can satisfy
        // `BarServiceImpl`'s call on `bar_foo()`.
        // in `bar_foo`, it converts to a reference to `Deps`,
        // which can call `foo_bar` from `FooService`
        // because the Container implements the `FooService` with call of `FooServiceImpl`'s `inj_ref` method
        // and AsRef<FooServiceState>(generated by derive(AsRef)), the foo_bar method can be called successfully!
        self.container.bar_foo();
        // Or (self.container as dyn BarService).bar_foo()
    }
}
```
