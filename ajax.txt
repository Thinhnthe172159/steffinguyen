//partial view
@model IEnumerable<newProject.Models.Item>

<table class="table">
    <thead>
        <tr>
            <th>@Html.DisplayNameFor(model => model.Name)</th>
            <th>@Html.DisplayNameFor(model => model.Description)</th>
            <th>@Html.DisplayNameFor(model => model.Type)</th>
            <th>@Html.DisplayNameFor(model => model.quantity)</th>
            <th></th>
        </tr>
    </thead>
    <tbody>
        @foreach (var item in Model)
        {
            <tr>
                <td>@Html.DisplayFor(modelItem => item.Name)</td>
                <td>@Html.DisplayFor(modelItem => item.Description)</td>
                <td>@Html.DisplayFor(modelItem => item.Type)</td>
                <td>@Html.DisplayFor(modelItem => item.quantity)</td>
                <td>
                    <a asp-action="Edit" asp-route-id="@item.Id">Edit</a> |
                    <a asp-action="Details" asp-route-id="@item.Id">Details</a> |
                    <a asp-action="Delete" asp-route-id="@item.Id">Delete</a>
                </td>
            </tr>
        }
    </tbody>
</table>

// index.cshtml
@model IEnumerable<newProject.Models.Item>

@{
    ViewData["Title"] = "Index";
}

<h1>Index</h1>

<p>
    <a asp-action="Create">Create New</a>
</p>

<div>
    <input type="text" id="search" placeholder="Search items..." class="form-control" />
</div>
<div id="response" class="alert" style="display:none;"></div>
<div class="container">
    <form id="myform">
        <div class="col-md-6">
            <lable class="text-danger">Entert Name item</lable>
            <input type="text" name="Name" class="form-control" />
        </div>
        <div class="col-md-6">
            <lable class="text-danger">Entert Description of item</lable>
            <input type="text" name="Description" class="form-control" />
        </div>
        <div class="col-md-6">
            <lable class="text-danger">Entert Item's Type</lable>
            <input type="text" name="Type" class="form-control" />
        </div>
        <div class="col-md-6">
            <lable class="text-danger">Entert quantity of item</lable>
            <input type="number" name="quantity" class="form-control" />
        </div>
        <div class="col-md-6">
            <button class="btn btn-primary" type="submit">Submit</button>
        </div>
    </form>
</div>
<div id="itemList">
    @await Html.PartialAsync("_itemPM", Model)
</div>

<script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
    
<script>
    $(document).ready(function () {
        $('#myform').submit(function (event) {
            event.preventDefault(); // Ngăn chặn hành vi mặc định của form

            $.ajax({
                type: 'POST',
                url: '@Url.Action("AddNew", "Items")', // Đường dẫn đến Action trong Controller
                data: $(this).serialize(), // Lấy dữ liệu từ form
                dataType: 'json', // Xác định kiểu dữ liệu trả về
                success: function (response) {
                    $('#response').removeClass('alert-danger').addClass('alert-success').html(response.message).show();
                    loadItemList();
                    // Nếu bạn muốn làm mới danh sách items, có thể gọi lại Ajax để lấy lại danh sách
                },
                error: function (xhr, status, error) {
                    $('#response').removeClass('alert-success').addClass('alert-danger').html("Đã xảy ra lỗi trong quá trình gửi dữ liệu.").show();
                    console.error(xhr.responseText);
                }
            });
        });
    });

    $('#myform').submit(function (event) {
        event.preventDefault(); // Ngăn chặn hành vi mặc định của form
        console.log("Form submitted"); // Kiểm tra xem hàm này có được gọi 
    });

    // hàm search sử dụng ajax
    $('#search').on('input', function () {
        var query = $(this).val();
        loadItemList(query);
    });

    // hàm tải lại danh sách từ partial view
    function loadItemList(query = '') {
        $.ajax({
            type: 'GET',
            url: '@Url.Action("GetItemList", "Items")',
            data: { search: query }, // Gửi từ khóa tìm kiếm
            success: function (response) {
                $('#itemList').html(response); // Cập nhật danh sách item
            }
        });
    }


</script>

//controller

using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.Rendering;
using Microsoft.DotNet.Scaffolding.Shared.Messaging;
using Microsoft.EntityFrameworkCore;
using newProject.Data;
using newProject.Models;

namespace newProject.Controllers
{
    public class ItemsController : Controller
    {
        private readonly ApplicationDbContext _context;

        public ItemsController(ApplicationDbContext context)
        {
            _context = context;
        }

        // GET: Items1
        public async Task<IActionResult> Index()
        {
            return View(await _context.items.ToListAsync());
        }

        [HttpGet]
        public IActionResult GetItemList(string? search)
        {
            var items = string.IsNullOrEmpty(search)
                ? _context.items.ToList()
                : _context.items.Where(i => i.Name.Contains(search) || i.Description.Contains(search)).ToList();

            return PartialView("_itemPM", items); // Trả về PartialView
        }

        // GET: Items1/Details/5
        public async Task<IActionResult> Details(int? id)
        {
            if (id == null)
            {
                return NotFound();
            }

            var item = await _context.items
                .FirstOrDefaultAsync(m => m.Id == id);
            if (item == null)
            {
                return NotFound();
            }

            return View(item);
        }

        // GET: Items1/Create
        public IActionResult Create()
        {
            return View();
        }

        // POST: Items1/Create
        // To protect from overposting attacks, enable the specific properties you want to bind to.
        // For more details, see http://go.microsoft.com/fwlink/?LinkId=317598.
        [HttpPost]
        [ValidateAntiForgeryToken]
        public async Task<IActionResult> Create([Bind("Id,Name,Description,Type,quantity")] Item item)
        {
            if (ModelState.IsValid)
            {
                _context.Add(item);
                await _context.SaveChangesAsync();
                return RedirectToAction(nameof(Index));
            }
            return View(item);
        }


        [HttpPost]
        public JsonResult AddNew(string Name, string? Description, string Type, int Quantity)
        {
            try
            {
                Item item = new Item(0, Name, Description, Type, Quantity);
                _context.items.Add(item);
                _context.SaveChanges();

                return Json(new { success = true, message = "Dữ liệu của bạn đã được lưu thành công!" });
            }
            catch (Exception e)
            {
                Console.WriteLine(e.ToString());
                return Json(new { success = false, message = "Dữ liệu đã lưu thất bại. Vui lòng thử lại!" });
            }
        }


        // GET: Items1/Edit/5
        public async Task<IActionResult> Edit(int? id)
        {
            if (id == null)
            {
                return NotFound();
            }

            var item = await _context.items.FindAsync(id);
            if (item == null)
            {
                return NotFound();
            }
            return View(item);
        }

        // POST: Items1/Edit/5
        // To protect from overposting attacks, enable the specific properties you want to bind to.
        // For more details, see http://go.microsoft.com/fwlink/?LinkId=317598.
        [HttpPost]
        [ValidateAntiForgeryToken]
        public async Task<IActionResult> Edit(int id, [Bind("Id,Name,Description,Type,quantity")] Item item)
        {
            if (id != item.Id)
            {
                return NotFound();
            }

            if (ModelState.IsValid)
            {
                try
                {
                    _context.Update(item);
                    await _context.SaveChangesAsync();
                }
                catch (DbUpdateConcurrencyException)
                {
                    if (!ItemExists(item.Id))
                    {
                        return NotFound();
                    }
                    else
                    {
                        throw;
                    }
                }
                return RedirectToAction(nameof(Index));
            }
            return View(item);
        }

        // GET: Items1/Delete/5
        public async Task<IActionResult> Delete(int? id)
        {
            if (id == null)
            {
                return NotFound();
            }

            var item = await _context.items
                .FirstOrDefaultAsync(m => m.Id == id);
            if (item == null)
            {
                return NotFound();
            }

            return View(item);
        }

        // POST: Items1/Delete/5
        [HttpPost, ActionName("Delete")]
        [ValidateAntiForgeryToken]
        public async Task<IActionResult> DeleteConfirmed(int id)
        {
            var item = await _context.items.FindAsync(id);
            if (item != null)
            {
                _context.items.Remove(item);
            }

            await _context.SaveChangesAsync();
            return RedirectToAction(nameof(Index));
        }

        private bool ItemExists(int id)
        {
            return _context.items.Any(e => e.Id == id);
        }
    }
}
