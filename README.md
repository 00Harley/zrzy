# 不造靠不靠谱涅拿chatgpt写的
## 1
blog/urls.py
```python
urlpatterns=[
#加一个
path('article/<int:article_id>/like/', views.article_like, name='article_like'),

]
```
## 2
blog/views.py
把原来的article_like函数换掉哩
```python
# 之前滴
# @login_required
# @require_POST
# # def like_article(request, article_id):
# #     article = get_object_or_404(Article, id=article_id)
# #     data = request.json()
# #
# #     if data.get('action') == 'like':
# #         article.likes.add(request.user)
# #     elif data.get('action') == 'unlike':
# #         article.likes.remove(request.user)
# #
# #     return JsonResponse({
# #         'status': 'success',
# #         'likes_count': article.likes.count()
# #     })

#改成酱紫了
from django.views.decorators.http import require_POST
@require_POST
@login_required
def article_like(request, article_id):
    print(f"[DEBUG] 当前用户: {request.user}")  # 检查用户是否切换成功
    if not request.user.is_authenticated:
        return JsonResponse({'status': 'error', 'message': '请登录'}, status=403)

    article = get_object_or_404(Article, id=article_id)
    user = request.user  # 明确使用当前用户

    if user in article.likes.all():
        article.likes.remove(user)
        is_liked = False
    else:
        article.likes.add(user)
        is_liked = True

    return JsonResponse({
        'likes_count': article.likes.count(),
        'is_liked': is_liked
    })

'''
清除缓存视图
'''
def clean_cache_view(request):
    cache.clear()
    return HttpResponse("ok")
```
# 3
templates/blog/tags/article_info.html
```python

<head>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
{#    Font Awesome是一个流行的图标库，提供了大量的矢量图标，可以方便地在网页中使用。#}
    <style>

'''
'''

#想不起来改没改了但是应该没改
        /* 点赞按钮 */
        .like-btn {
            background: none;
            border: none;
            cursor: pointer;
            transition: all 0.3s;
            padding: 5px 10px;
            border-radius: 20px;
            display: flex;
            align-items: center;
        }

        .like-btn:hover {
            background: rgba(67, 97, 238, 0.1);
        }

        .like-btn:hover i {
            transform: scale(1.1);  /* 悬停时轻微放大 */
        }

        .like-btn i.liked {
            color: #ff4757;
        }

'''
'''
    </style>
</head>
```

templates/blog/tags/article_info.html的最后面加
```python
<script>
// 事件委托监听所有点赞按钮（包括动态加载的）
document.body.addEventListener('click', async (e) => {
    const button = e.target.closest('.like-btn');
    if (!button) return;

    e.preventDefault();
    const articleId = button.dataset.articleId;
    const url = `/article/${articleId}/like/`;
    const csrftoken = document.querySelector('meta[name="csrf-token"]').content;

    try {
        const response = await fetch(url, {
            method: 'POST',
            headers: {
                'X-CSRFToken': csrftoken,
                'Content-Type': 'application/json',
            },
            credentials: 'same-origin'  // 确保携带 Cookie
        });

        if (response.status === 403) {
            window.location.href = '/login/';  // 直接跳转登录页
            return;
        }

        const data = await response.json();
        const likesCountSpan = document.getElementById(`likes-count-${articleId}`);
        const icon = button.querySelector('i');

        likesCountSpan.textContent = data.likes_count;
        icon.className = data.is_liked ? 'fas fa-thumbs-up liked' : 'far fa-thumbs-up';
    } catch (error) {
        console.error('Error:', error);
    }
});
</script>


<!-- 在头部添加 CSRF Token -->
<meta name="csrf-token" content="{{ csrf_token }}">

```

#有个小问题就是不造为什么在首页缩略图里点赞好像一次会调用4次点赞函数？有时候会抽

# 4
blog/models.py
```python
class Article(BaseModel):
    # 新增点赞数和点赞用户字段
    likes = models.ManyToManyField(
        get_user_model(),
        related_name='liked_articles',
        blank=True
    )
```
