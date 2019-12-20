---
title: Django 与 celery 出现幻读
date: 2019-12-18 23:11:43
tags: [ 'Django', 'Python' ]
categories: "Python"
---

> ** 前言 **
　因为 Jenkins 的没有办法满足我们的优化场景，所以我们自己写了一个平台来处理 gitlab 的 webhook 然后 build image，为了快速上线不写前端我们用的 django admin 来做，正因为偷懒就踩了这个坑，第一次遇到幻读。


## 背景

## 问题

在用户使用 admin 中增加记录的方式来触发构建镜像时会出现 celery 的 worker 报错：

```python
ci.models.DoesNotExist: TestBuild matching query does not exist.
```

但是在给 celery 发送任务之前我们是执行过 `save()` 方法，并拿到新增的 id 作为参数传给 task 的，按道理说不应该出现这样的问题才对。

## 解决

### 原因

有了新增 id，但是数据库中就是查不到，只有一个可能就是这个事务并没有完成（django 是 read-committed 的隔离级别），但是 django1.4 是之后 `autocommit` 是默认开启的，那就很奇怪了。

只能返回查看 admin 的代码是否在哪里关掉了 `autocommit` 了，首先是进入增加记录的页面 `xxx.xxx.com/app/model/add`，我们查看一下源码做了什么操作：

- 路由相关的请求到对应的方法

```python
def wrap(view):
    def wrapper(*args, **kwargs):
        return self.admin_site.admin_view(view)(*args, **kwargs)
    wrapper.model_admin = self
    return update_wrapper(wrapper, view)

info = self.model._meta.app_label, self.model._meta.model_name

urlpatterns = [
    path('', wrap(self.changelist_view), name='%s_%s_changelist' % info),
    path('add/', wrap(self.add_view), name='%s_%s_add' % info),
    path('autocomplete/', wrap(self.autocomplete_view), name='%s_%s_autocomplete' % info),
    path('<path:object_id>/history/', wrap(self.history_view), name='%s_%s_history' % info),
    path('<path:object_id>/delete/', wrap(self.delete_view), name='%s_%s_delete' % info),
    path('<path:object_id>/change/', wrap(self.change_view), name='%s_%s_change' % info),
    # For backwards compatibility (was the change url before 1.9)
    path('<path:object_id>/', wrap(RedirectView.as_view(
        pattern_name='%s:%s_%s_change' % ((self.admin_site.name,) + info)
    ))),
]
return urlpatterns
```

- 查看 `add_view` 的处理

```python
def add_view(self, request, form_url='', extra_context=None):
    return self.changeform_view(request, None, form_url, extra_context)
```

- 查看 `changeform_view` 的处理

```python
@csrf_protect_m
def changeform_view(self, request, object_id=None, form_url='', extra_context=None):
    with transaction.atomic(using=router.db_for_write(self.model)):
        return self._changeform_view(request, object_id, form_url, extra_context)
```

- 可以看到开启了一个事务，并继续给 `_changeform_view` 处理，我们先看一下这里面的处理，再看 `transaction.atomic` 的操作

```python
def _changeform_view(self, request, object_id, form_url, extra_context):
    to_field = request.POST.get(TO_FIELD_VAR, request.GET.get(TO_FIELD_VAR))
    if to_field and not self.to_field_allowed(request, to_field):
        raise DisallowedModelAdminToField("The field %s cannot be referenced." % to_field)

    model = self.model
    opts = model._meta

    if request.method == 'POST' and '_saveasnew' in request.POST:
        object_id = None

    add = object_id is None

    if add:
        if not self.has_add_permission(request):
            raise PermissionDenied
        obj = None

    else:
        obj = self.get_object(request, unquote(object_id), to_field)

        if not self.has_change_permission(request, obj):
            raise PermissionDenied

        if obj is None:
            return self._get_obj_does_not_exist_redirect(request, opts, object_id)

    ModelForm = self.get_form(request, obj)
    if request.method == 'POST':
        form = ModelForm(request.POST, request.FILES, instance=obj)
        if form.is_valid():
            form_validated = True
            new_object = self.save_form(request, form, change=not add)
        else:
            form_validated = False
            new_object = form.instance
        formsets, inline_instances = self._create_formsets(request, new_object, change=not add)
        if all_valid(formsets) and form_validated:
            self.save_model(request, new_object, form, not add)
            self.save_related(request, form, formsets, not add)
            change_message = self.construct_change_message(request, form, formsets, add)
            if add:
                self.log_addition(request, new_object, change_message)
                return self.response_add(request, new_object)
            else:
                self.log_change(request, new_object, change_message)
                return self.response_change(request, new_object)
        else:
            form_validated = False
    else:
        if add:
            initial = self.get_changeform_initial_data(request)
            form = ModelForm(initial=initial)
            formsets, inline_instances = self._create_formsets(request, form.instance, change=False)
        else:
            form = ModelForm(instance=obj)
            formsets, inline_instances = self._create_formsets(request, obj, change=True)

    adminForm = helpers.AdminForm(
        form,
        list(self.get_fieldsets(request, obj)),
        self.get_prepopulated_fields(request, obj),
        self.get_readonly_fields(request, obj),
        model_admin=self)
    media = self.media + adminForm.media

    inline_formsets = self.get_inline_formsets(request, formsets, inline_instances, obj)
    for inline_formset in inline_formsets:
        media = media + inline_formset.media

    context = dict(
        self.admin_site.each_context(request),
        title=(_('Add %s') if add else _('Change %s')) % opts.verbose_name,
        adminform=adminForm,
        object_id=object_id,
        original=obj,
        is_popup=(IS_POPUP_VAR in request.POST or
                  IS_POPUP_VAR in request.GET),
        to_field=to_field,
        media=media,
        inline_admin_formsets=inline_formsets,
        errors=helpers.AdminErrorList(form, formsets),
        preserved_filters=self.get_preserved_filters(request),
    )

    # Hide the "Save" and "Save and continue" buttons if "Save as New" was
    # previously chosen to prevent the interface from getting confusing.
    if request.method == 'POST' and not form_validated and "_saveasnew" in request.POST:
        context['show_save'] = False
        context['show_save_and_continue'] = False
        # Use the change template instead of the add template.
        add = False

    context.update(extra_context or {})

    return self.render_change_form(request, context, add=add, change=not add, obj=obj, form_url=form_url)
```

从上述的代码中我们可以看到其实这个方法就是通过 `get_form` 与 `render_change_form` 为我们展示了那个我们可以随意编辑增加记录的页面

- 若是 `Get` 方法就是我们首次第一访问主要就是获取字段、鉴权、初始化表单等操作
- 若是 `POST` 方法便是我们在点击了 `save` 的时候提交了该表单，而在 `POST` 方式处理里面调用了 `save_model`, 而我们的触发任务构建便是写在了 `save_model` 中

```python
# 我们重写的内容
def save_model(self, request, obj, form, change):
    obj.trigger = request.user  # 触发人
    # 在重写 save_model 之前，只有一个操作就是 obj.save()
    super().save_model(request, obj, form, change)
    if obj.task is None:
        # start_build 方法里面我们将新增的 id 作为参数传给 celery task
        obj.start_build()
```

我们明明在 `start_build` 之前 `save()` 为什么数据库里面读不到呢？我们回过头来看一下 `transaction.atomic`:

```
class Atomic(ContextDecorator):
    def __init__(self, using, savepoint):
        self.using = using
        self.savepoint = savepoint

    def __enter__(self):
        connection = get_connection(self.using)

        if not connection.in_atomic_block:
            connection.commit_on_exit = True
            connection.needs_rollback = False
            if not connection.get_autocommit():
                # Some database adapters (namely sqlite3) don't handle
                # transactions and savepoints properly when autocommit is off.
                # Turning autocommit back on isn't an option; it would trigger
                # a premature commit. Give up if that happens.
                if connection.features.autocommits_when_autocommit_is_off:
                    raise TransactionManagementError(
                        "Your database backend doesn't behave properly when "
                        "autocommit is off. Turn it on before using 'atomic'.")
                # Pretend we're already in an atomic block to bypass the code
                # that disables autocommit to enter a transaction, and make a
                # note to deal with this case in __exit__.
                connection.in_atomic_block = True
                connection.commit_on_exit = False

        if connection.in_atomic_block:
            # We're already in a transaction; create a savepoint, unless we
            # were told not to or we're already waiting for a rollback. The
            # second condition avoids creating useless savepoints and prevents
            # overwriting needs_rollback until the rollback is performed.
            if self.savepoint and not connection.needs_rollback:
                sid = connection.savepoint()
                connection.savepoint_ids.append(sid)
            else:
                connection.savepoint_ids.append(None)
        else:
            connection.set_autocommit(False, force_begin_transaction_with_broken_autocommit=True)
            connection.in_atomic_block = True

    def __exit__(self, exc_type, exc_value, traceback):
        connection = get_connection(self.using)

        if connection.savepoint_ids:
            sid = connection.savepoint_ids.pop()
        else:
            # Prematurely unset this flag to allow using commit or rollback.
            connection.in_atomic_block = False

        try:
            if connection.closed_in_transaction:
                # The database will perform a rollback by itself.
                # Wait until we exit the outermost block.
                pass

            elif exc_type is None and not connection.needs_rollback:
                if connection.in_atomic_block:
                    # Release savepoint if there is one
                    if sid is not None:
                        try:
                            connection.savepoint_commit(sid)
                        except DatabaseError:
                            try:
                                connection.savepoint_rollback(sid)
                                # The savepoint won't be reused. Release it to
                                # minimize overhead for the database server.
                                connection.savepoint_commit(sid)
                            except Error:
                                # If rolling back to a savepoint fails, mark for
                                # rollback at a higher level and avoid shadowing
                                # the original exception.
                                connection.needs_rollback = True
                            raise
                else:
                    # Commit transaction
                    try:
                        connection.commit()
                    except DatabaseError:
                        try:
                            connection.rollback()
                        except Error:
                            # An error during rollback means that something
                            # went wrong with the connection. Drop it.
                            connection.close()
                        raise
            else:
                # This flag will be set to True again if there isn't a savepoint
                # allowing to perform the rollback at this level.
                connection.needs_rollback = False
                if connection.in_atomic_block:
                    # Roll back to savepoint if there is one, mark for rollback
                    # otherwise.
                    if sid is None:
                        connection.needs_rollback = True
                    else:
                        try:
                            connection.savepoint_rollback(sid)
                            # The savepoint won't be reused. Release it to
                            # minimize overhead for the database server.
                            connection.savepoint_commit(sid)
                        except Error:
                            # If rolling back to a savepoint fails, mark for
                            # rollback at a higher level and avoid shadowing
                            # the original exception.
                            connection.needs_rollback = True
                else:
                    # Roll back transaction
                    try:
                        connection.rollback()
                    except Error:
                        # An error during rollback means that something
                        # went wrong with the connection. Drop it.
                        connection.close()

        finally:
            # Outermost block exit when autocommit was enabled.
            if not connection.in_atomic_block:
                if connection.closed_in_transaction:
                    connection.connection = None
                else:
                    connection.set_autocommit(True)
            # Outermost block exit when autocommit was disabled.
            elif not connection.savepoint_ids and not connection.commit_on_exit:
                if connection.closed_in_transaction:
                    connection.connection = None
                else:
                    connection.in_atomic_block = False

def atomic(using=None, savepoint=True):
    # Bare decorator: @atomic -- although the first argument is called
    # `using`, it's actually the function being decorated.
    if callable(using):
        return Atomic(DEFAULT_DB_ALIAS, savepoint)(using)
    # Decorator: @atomic(...) or context manager: with atomic(...): ...
    else:
        return Atomic(using, savepoint)
```

从中我们看到了其实我们这个处理都还在 `with transaction.atomic()` 中没有结束，也就是进入了 Atomic 的 `__enter__` 中，所以我们事务其实一直没有 commit，而且其中 `save_model` 之前的 `save_form` 方法是传了 `commit=False` 的参数

```python
def save_form(self, request, form, change):
    """
    Given a ModelForm return an unsaved instance. ``change`` is True if
    the object is being changed, and False if it's being added.
    """
    return form.save(commit=False)
```

所以就是这样的原因导致了我们的事务没有关闭，就给 celery 发任务的，立马 worker 就会接收到并处理其中的任务，而在这边事务结束之前，worker 查数据库就是查不到这个记录

### 解决

上面我们知道了问题的所在，解决方案就是在发任务之前我们需要将之前没有完成的事务结束，这样 celery 再通过数据库查的时候就不会出现幻读了：

```python
def save_model(self, request, obj, form, change):
    obj.trigger = request.user
    super().save_model(request, obj, form, change)
    if obj.task is None:
        # commit 上一次的没有结束的事务，在开始 start_build(这里面会给 celery 发任务)
        transaction.on_commit(
            lambda: obj.start_build())
```

## 参考

- https://medium.com/hypertrack/dealing-with-database-transactions-in-django-celery-eac351d52f5f
- https://blog.csdn.net/ikuboo/article/details/81709888
