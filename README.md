
--[[
DELTA EXPLORER - Luau
Explorer personalizado para o cliente Roblox.
]]

local Players = game:GetService("Players")
local UIS = game:GetService("UserInputService")
local HttpService = game:GetService("HttpService")
local LocalPlayer = Players.LocalPlayer

local requestClipboard = setclipboard or toclipboard
local getClipboard = getclipboard

local parentGui = game:GetService("CoreGui")
pcall(function()
    if gethui then parentGui = gethui() end
end)

pcall(function()
    local old = parentGui:FindFirstChild("DeltaExplorerFunctional")
    if old then old:Destroy() end
end)

local gui = Instance.new("ScreenGui")
gui.Name = "DeltaExplorerFunctional"
gui.ResetOnSpawn = false
gui.ZIndexBehavior = Enum.ZIndexBehavior.Sibling
gui.Parent = parentGui

local COLORS = {
    bg = Color3.fromRGB(24, 26, 32),
    panel = Color3.fromRGB(31, 34, 42),
    header = Color3.fromRGB(39, 43, 53),
    row = Color3.fromRGB(34, 37, 46),
    rowAlt = Color3.fromRGB(29, 32, 40),
    selected = Color3.fromRGB(48, 91, 151),
    hover = Color3.fromRGB(45, 49, 60),
    text = Color3.fromRGB(232, 235, 242),
    muted = Color3.fromRGB(157, 165, 180),
    accent = Color3.fromRGB(65, 132, 225),
    danger = Color3.fromRGB(190, 62, 70),
    green = Color3.fromRGB(46, 143, 91),
    border = Color3.fromRGB(55, 60, 72),
}

local function create(className, props, parent)
    local obj = Instance.new(className)
    for k, v in pairs(props or {}) do
        pcall(function()
            obj[k] = v
        end)
    end
    obj.Parent = parent
    return obj
end

local function corner(obj, radius)
    create("UICorner", {
        CornerRadius = UDim.new(0, radius or 5)
    }, obj)
end

local function stroke(obj, color, thickness)
    create("UIStroke", {
        Color = color or COLORS.border,
        Thickness = thickness or 1,
        Transparency = 0.15,
    }, obj)
end

local function label(parent, text, size, pos, fontSize, color)
    return create("TextLabel", {
        BackgroundTransparency = 1,
        Text = text or "",
        TextColor3 = color or COLORS.text,
        TextSize = fontSize or 13,
        Font = Enum.Font.Code,
        TextXAlignment = Enum.TextXAlignment.Left,
        TextYAlignment = Enum.TextYAlignment.Center,
        Size = size,
        Position = pos,
        TextTruncate = Enum.TextTruncate.AtEnd,
    }, parent)
end

local function button(parent, text, size, pos, callback, color)
    local b = create("TextButton", {
        Text = text,
        Size = size,
        Position = pos,
        BackgroundColor3 = color or COLORS.header,
        TextColor3 = COLORS.text,
        TextSize = 12,
        Font = Enum.Font.Code,
        AutoButtonColor = true,
        BorderSizePixel = 0,
    }, parent)
    corner(b, 5)
    if callback then
        b.MouseButton1Click:Connect(callback)
    end
    return b
end

local main = create("Frame", {
    Name = "Main",
    Size = UDim2.new(0, 850, 0, 570),
    Position = UDim2.new(0.5, -425, 0.5, -285),
    BackgroundColor3 = COLORS.bg,
    BorderSizePixel = 0,
    Active = true,
}, gui)

corner(main, 9)
stroke(main)

local topbar = create("Frame", {
    Size = UDim2.new(1, 0, 0, 38),
    BackgroundColor3 = COLORS.header,
    BorderSizePixel = 0,
}, main)

corner(topbar, 9)

label(topbar, "DELTA EXPLORER",
    UDim2.new(0, 220, 1, 0),
    UDim2.new(0, 12, 0, 0), 15)

label(topbar, "Cliente • hierarquia local",
    UDim2.new(0, 260, 1, 0),
    UDim2.new(0, 180, 0, 0), 11, COLORS.muted)

local closeButton = button(
    topbar, "X",
    UDim2.new(0, 32, 0, 28),
    UDim2.new(1, -36, 0, 5),
    function()
        gui:Destroy()
    end,
    COLORS.danger
)

-- Arrastar janela com mouse ou toque
local dragging, dragStart, startPos

topbar.InputBegan:Connect(function(input)
    if input.UserInputType == Enum.UserInputType.MouseButton1
    or input.UserInputType == Enum.UserInputType.Touch then
        dragging = true
        dragStart = input.Position
        startPos = main.Position

        input.Changed:Connect(function()
            if input.UserInputState == Enum.UserInputState.End then
                dragging = false
            end
        end)
    end
end)

UIS.InputChanged:Connect(function(input)
    if dragging and (
        input.UserInputType == Enum.UserInputType.MouseMovement
        or input.UserInputType == Enum.UserInputType.Touch
    ) then
        local delta = input.Position - dragStart
        main.Position = UDim2.new(
            startPos.X.Scale,
            startPos.X.Offset + delta.X,
            startPos.Y.Scale,
            startPos.Y.Offset + delta.Y
        )
    end
end)

local searchBox = create("TextBox", {
    Size = UDim2.new(1, -290, 0, 30),
    Position = UDim2.new(0, 10, 0, 46),
    BackgroundColor3 = COLORS.panel,
    TextColor3 = COLORS.text,
    PlaceholderColor3 = COLORS.muted,
    PlaceholderText = "Pesquisar instancia pelo nome...",
    Text = "",
    ClearTextOnFocus = false,
    TextSize = 13,
    Font = Enum.Font.Code,
    BorderSizePixel = 0,
}, main)

corner(searchBox, 5)

local refreshButton = button(
    main, "Atualizar",
    UDim2.new(0, 80, 0, 30),
    UDim2.new(1, -270, 0, 46)
)

local expandButton = button(
    main, "Expandir",
    UDim2.new(0, 80, 0, 30),
    UDim2.new(1, -185, 0, 46)
)

local collapseButton = button(
    main, "Recolher",
    UDim2.new(0, 80, 0, 30),
    UDim2.new(1, -100, 0, 46)
)

local status = label(
    main,
    "Pronto. Selecione uma instancia.",
    UDim2.new(1, -20, 0, 20),
    UDim2.new(0, 10, 0, 80),
    11,
    COLORS.muted
)

local explorerPane = create("Frame", {
    Position = UDim2.new(0, 10, 0, 105),
    Size = UDim2.new(0, 430, 1, -115),
    BackgroundColor3 = COLORS.panel,
    BorderSizePixel = 0,
}, main)

corner(explorerPane, 6)
stroke(explorerPane)

label(explorerPane, "EXPLORER",
    UDim2.new(1, -12, 0, 25),
    UDim2.new(0, 8, 0, 0), 12, COLORS.muted)

local treeScroll = create("ScrollingFrame", {
    Position = UDim2.new(0, 4, 0, 28),
    Size = UDim2.new(1, -8, 1, -32),
    BackgroundTransparency = 1,
    BorderSizePixel = 0,
    ScrollBarThickness = 6,
    CanvasSize = UDim2.new(0, 0, 0, 0),
    AutomaticCanvasSize = Enum.AutomaticSize.Y,
}, explorerPane)

local treeLayout = create("UIListLayout", {
    SortOrder = Enum.SortOrder.LayoutOrder,
    Padding = UDim.new(0, 1),
}, treeScroll)

local propsPane = create("Frame", {
    Position = UDim2.new(0, 450, 0, 105),
    Size = UDim2.new(1, -460, 1, -115),
    BackgroundColor3 = COLORS.panel,
    BorderSizePixel = 0,
}, main)

corner(propsPane, 6)
stroke(propsPane)

label(propsPane, "PROPERTIES",
    UDim2.new(1, -12, 0, 25),
    UDim2.new(0, 8, 0, 0), 12, COLORS.muted)

local propsScroll = create("ScrollingFrame", {
    Position = UDim2.new(0, 5, 0, 28),
    Size = UDim2.new(1, -10, 1, -33),
    BackgroundTransparency = 1,
    BorderSizePixel = 0,
    ScrollBarThickness = 5,
    CanvasSize = UDim2.new(0, 0, 0, 0),
    AutomaticCanvasSize = Enum.AutomaticSize.Y,
}, propsPane)

create("UIListLayout", {
    SortOrder = Enum.SortOrder.LayoutOrder,
    Padding = UDim.new(0, 3),
}, propsScroll)

local expanded = setmetatable({}, {__mode = "k"})
local selected = {}
local selectedSet = {}
local visibleInstances = {}
local rowForInstance = {}
local anchorSelection = nil
local clipboardInstances = {}
local clipboardMode = "copy"
local lastClickInstance = nil
local propertyRefreshToken = 0

local function getRoots()
    local roots = {}

    pcall(function()
        table.insert(roots, game:GetService("Workspace"))
    end)

    for _, serviceName in ipairs({
        "Players", "Lighting", "ReplicatedFirst",
        "ReplicatedStorage", "ServerStorage",
        "ServerScriptService", "StarterGui",
        "StarterPack", "StarterPlayer", "Teams",
        "SoundService", "TextChatService",
        "MaterialService"
    }) do
        local success, service = pcall(function()
            return game:GetService(serviceName)
        end)

        if success and service then
            table.insert(roots, service)
        end
    end

    if LocalPlayer then
        table.insert(roots, LocalPlayer)
        local pg = LocalPlayer:FindFirstChildOfClass("PlayerGui")
        if pg then
            table.insert(roots, pg)
        end
    end

    local unique, seen = {}, {}
    for _, obj in ipairs(roots) do
        if obj and not seen[obj] then
            seen[obj] = true
            table.insert(unique, obj)
        end
    end

    return unique
end

local function clearGuiChildren(parent, exceptLayout)
    for _, child in ipairs(parent:GetChildren()) do
        if child ~= exceptLayout
        and not child:IsA("UICorner")
        and not child:IsA("UIStroke") then
            child:Destroy()
        end
    end
end

local function isSelected(obj)
    return selectedSet[obj] == true
end

local function setSelection(list)
    selected = {}
    selectedSet = {}

    for _, obj in ipairs(list) do
        if typeof(obj) == "Instance" and obj.Parent then
            if not selectedSet[obj] then
                selectedSet[obj] = true
                table.insert(selected, obj)
            end
        end
    end

    anchorSelection = selected[#selected]
end

local function selectOne(obj, additive, range)
    if range and anchorSelection then
        local startIndex, endIndex

        for i, inst in ipairs(visibleInstances) do
            if inst == anchorSelection then startIndex = i end
            if inst == obj then endIndex = i end
        end

        if startIndex and endIndex then
            local newList = {}

            if additive then
                for _, old in ipairs(selected) do
                    table.insert(newList, old)
                end
            end

            for i = math.min(startIndex, endIndex),
                math.max(startIndex, endIndex) do
                table.insert(newList, visibleInstances[i])
            end

            setSelection(newList)
            return
        end
    end

    if additive then
        local newList = {}

        for _, old in ipairs(selected) do
            if old ~= obj then
                table.insert(newList, old)
            end
        end

        if not selectedSet[obj] then
            table.insert(newList, obj)
        end

        setSelection(newList)
    else
        setSelection({obj})
    end
end

local function safeGet(obj, prop)
    local ok, value = pcall(function()
        return obj[prop]
    end)

    if ok then
        return true, value
    end

    return false, nil
end

local function valueString(value)
    local t = typeof(value)

    if t == "Instance" then
        return value:GetFullName()
    elseif t == "Color3" then
        return string.format("%.3f, %.3f, %.3f",
            value.R, value.G, value.B)
    elseif t == "Vector3" then
        return string.format("%.3f, %.3f, %.3f",
            value.X, value.Y, value.Z)
    elseif t == "Vector2" then
        return string.format("%.3f, %.3f",
            value.X, value.Y)
    elseif t == "CFrame" then
        local p = value.Position
        return string.format("%.2f, %.2f, %.2f",
            p.X, p.Y, p.Z)
    elseif t == "BrickColor" then
        return value.Name
    elseif t == "EnumItem" then
        return tostring(value)
    elseif t == "table" then
        return "{table}"
    end

    return tostring(value)
end

local function setStatus(text)
    status.Text = tostring(text)
end

local function showProperties()
    propertyRefreshToken += 1
    local token = propertyRefreshToken

    clearGuiChildren(propsScroll)

    if #selected == 0 then
        label(propsScroll, "Nenhuma instancia selecionada.",
            UDim2.new(1, -8, 0, 30),
            UDim2.new(), 12, COLORS.muted)
        return
    end

    local obj = selected[#selected]

    label(propsScroll, obj.Name,
        UDim2.new(1, -8, 0, 24),
        UDim2.new(), 13, COLORS.text).Font = Enum.Font.Code

    label(propsScroll,
        obj.ClassName .. "  •  " .. tostring(#selected) .. " selecionada(s)",
        UDim2.new(1, -8, 0, 22),
        UDim2.new(), 11, COLORS.muted)

    local properties = {
        "Name", "ClassName", "Archivable", "Parent",
        "Anchored", "CanCollide", "CanTouch", "CanQuery",
        "Transparency", "LocalTransparencyModifier", "Reflectance",
        "Color", "Material", "Size", "Position", "Orientation",
        "CFrame", "Massless", "CastShadow",
        "Visible", "Enabled", "Text", "TextColor3",
        "BackgroundColor3", "Image", "Texture", "MeshId",
        "TextureID", "Value", "WalkSpeed", "JumpPower",
        "Health", "MaxHealth", "Volume", "PlaybackSpeed",
        "Looped", "Playing", "Disabled", "Source"
    }

    local seen = {}
    local order = 0

    for _, prop in ipairs(properties) do
        if not seen[prop] then
            seen[prop] = true

            local ok, value = safeGet(obj, prop)

            if ok then
                order += 1

                local row = create("Frame", {
                    Size = UDim2.new(1, -2, 0, 42),
                    BackgroundColor3 = order % 2 == 0
                        and COLORS.rowAlt or COLORS.row,
                    BorderSizePixel = 0,
                    LayoutOrder = order,
                }, propsScroll)

                corner(row, 4)

                label(row, prop,
                    UDim2.new(0.42, -5, 1, 0),
                    UDim2.new(0, 6, 0, 0),
                    11, COLORS.muted)

                local edit = create("TextBox", {
                    Size = UDim2.new(0.58, -8, 1, -6),
                    Position = UDim2.new(0.42, 0, 0, 3),
                    BackgroundColor3 = COLORS.header,
                    TextColor3 = COLORS.text,
                    PlaceholderColor3 = COLORS.muted,
                    Text = valueString(value),
                    ClearTextOnFocus = false,
                    TextSize = 11,
                    Font = Enum.Font.Code,
                    TextXAlignment = Enum.TextXAlignment.Left,
                    BorderSizePixel = 0,
                }, row)

                corner(edit, 4)

                edit.FocusLost:Connect(function(enter)
                    if not enter or token ~= propertyRefreshToken then
                        return
                    end

                    if prop == "ClassName" or prop == "Parent"
                    or prop == "CFrame" or prop == "Source"
                    or prop == "MeshId" or prop == "TextureID" then
                        setStatus(prop .. " nao e editavel por este painel.")
                        return
                    end

                    local text = edit.Text
                    local currentOk, current = safeGet(obj, prop)
                    if not currentOk then return end

                    local typ = typeof(current)
                    local newValue = nil

                    if typ == "string" then
                        newValue = text

                    elseif typ == "boolean" then
                        local lower = string.lower(text)

                        if lower == "true" then
                            newValue = true
                        elseif lower == "false" then
                            newValue = false
                        else
                            setStatus("Use true ou false.")
                            return
                        end

                    elseif typ == "number" then
                        newValue = tonumber(text)

                        if not newValue then
                            setStatus("Numero invalido.")
                            return
                        end

                    elseif typ == "Color3" then
                        local r, g, b = text:match(
                            "^%s*([%d%.]+)%s*,%s*([%d%.]+)%s*,%s*([%d%.]+)%s*$"
                        )

                        if r and g and b then
                            r, g, b = tonumber(r), tonumber(g), tonumber(b)

                            if r and g and b then
                                if math.max(r, g, b) > 1 then
                                    r, g, b = r/255, g/255, b/255
                                end

                                newValue = Color3.new(
                                    math.clamp(r, 0, 1),
                                    math.clamp(g, 0, 1),
                                    math.clamp(b, 0, 1)
                                )
                            end
                        end

                        if not newValue then
                            setStatus("Cor: R, G, B (0-1 ou 0-255).")
                            return
                        end

                    elseif typ == "Vector3" then
                        local x, y, z = text:match(
                            "^%s*([%-%d%.]+)%s*,%s*([%-%d%.]+)%s*,%s*([%-%d%.]+)%s*$"
                        )

                        if x and y and z then
                            newValue = Vector3.new(
                                tonumber(x), tonumber(y), tonumber(z)
                            )
                        end

                        if not newValue then
                            setStatus("Vector3: X, Y, Z.")
                            return
                        end

                    elseif typ == "EnumItem" then
                        local enumType = tostring(current):match(
                            "^Enum%.([^%.]+)%."
                        )

                        if enumType and Enum[enumType] then
                            local enumName = text:match("([^%.]+)$") or text

                            local success, enumValue = pcall(function()
                                return Enum[enumType][enumName]
                            end)

                            if success then
                                newValue = enumValue
                            end
                        end

                        if not newValue then
                            setStatus("Valor de Enum invalido.")
                            return
                        end

                    else
                        setStatus("Tipo nao editavel por este painel: " .. typ)
                        return
                    end

                    local success, err = pcall(function()
                        for _, target in ipairs(selected) do
                            target[prop] = newValue
                        end
                    end)

                    if success then
                        setStatus("Propriedade " .. prop .. " atualizada.")
                        showProperties()
                    else
                        setStatus("Falha ao alterar " .. prop .. ": " .. tostring(err))
                    end
                end)
            end
        end
    end
end

local function descendantsOf(obj)
    local result = {}
    local ok, descendants = pcall(function()
        return obj:GetDescendants()
    end)
    if ok then result = descendants end
    return result
end

local function isExpandable(obj)
    local ok, children = pcall(function()
        return obj:GetChildren()
    end)
    return ok and #children > 0
end

local function buildTree()
    clearGuiChildren(treeScroll, treeLayout)
    visibleInstances = {}
    rowForInstance = {}

    local query = string.lower(searchBox.Text or "")
    local roots = getRoots()
    local layoutOrder = 0

    local function hasMatchInTree(obj)
        if query == "" then return true end

        local ok, name = pcall(function()
            return string.lower(obj.Name)
        end)

        if ok and string.find(name, query, 1, true) then
            return true
        end

        for _, child in ipairs(descendantsOf(obj)) do
            local childOk, childName = pcall(function()
                return string.lower(child.Name)
            end)

            if childOk and string.find(childName, query, 1, true) then
                return true
            end
        end

        return false
    end

    local function draw(obj, depth)
        if not obj or not hasMatchInTree(obj) then return end

        layoutOrder += 1
        table.insert(visibleInstances, obj)

        local row = create("Frame", {
            Size = UDim2.new(1, -4, 0, 24),
            BackgroundColor3 = isSelected(obj) and COLORS.selected
                or (layoutOrder % 2 == 0 and COLORS.rowAlt or COLORS.row),
            BorderSizePixel = 0,
            LayoutOrder = layoutOrder,
        }, treeScroll)

        rowForInstance[obj] = row

        local arrow = ""
        if isExpandable(obj) then
            arrow = expanded[obj] and "▼" or "▶"
        end

        local indent = math.min(depth * 13, 150)

        local expand = create("TextButton", {
            Size = UDim2.new(0, 18, 1, 0),
            Position = UDim2.new(0, indent, 0, 0),
            BackgroundTransparency = 1,
            Text = arrow,
            TextColor3 = COLORS.muted,
            TextSize = 10,
            Font = Enum.Font.Code,
        }, row)

        expand.MouseButton1Click:Connect(function()
            expanded[obj] = not expanded[obj]
            buildTree()
        end)

        local iconText = obj:IsA("BasePart") and "◆"
            or obj:IsA("Model") and "▣"
            or obj:IsA("LuaSourceContainer") and "ƒ"
            or obj:IsA("ValueBase") and "◇"
            or "•"

        local nameLabel = create("TextButton", {
            Size = UDim2.new(1, -(indent + 23), 1, 0),
            Position = UDim2.new(0, indent + 19, 0, 0),
            BackgroundTransparency = 1,
            Text = iconText .. "  " .. obj.Name .. "  [" .. obj.ClassName .. "]",
            TextColor3 = isSelected(obj) and Color3.new(1, 1, 1)
                or COLORS.text,
            TextSize = 11,
            Font = Enum.Font.Code,
            TextXAlignment = Enum.TextXAlignment.Left,
            TextTruncate = Enum.TextTruncate.AtEnd,
        }, row)

        nameLabel.MouseButton1Click:Connect(function()
            local ctrl = UIS:IsKeyDown(Enum.KeyCode.LeftControl)
                or UIS:IsKeyDown(Enum.KeyCode.RightControl)
            local shift = UIS:IsKeyDown(Enum.KeyCode.LeftShift)
                or UIS:IsKeyDown(Enum.KeyCode.RightShift)

            selectOne(obj, ctrl, shift)
            buildTree()
            showProperties()
            lastClickInstance = obj
        end)

        nameLabel.MouseButton2Click:Connect(function()
            selectOne(obj, false, false)
            buildTree()
            showProperties()

            local mouse = UIS:GetMouseLocation()
            local camera = workspace.CurrentCamera
            if not camera then return end

            local menu = create("Frame", {
                Size = UDim2.new(0, 180, 0, 242),
                Position = UDim2.new(
                    0,
                    math.clamp(mouse.X, 5, camera.ViewportSize.X - 185),
                    0,
                    math.clamp(mouse.Y, 5, camera.ViewportSize.Y - 247)
                ),
                BackgroundColor3 = COLORS.header,
                BorderSizePixel = 0,
                ZIndex = 50,
            }, gui)

            corner(menu, 6)
            stroke(menu)

            local actions = {
                {"Copiar", function()
                    clipboardInstances = {}
                    for _, v in ipairs(selected) do
                        table.insert(clipboardInstances, v)
                    end
                    clipboardMode = "copy"
                    setStatus("Instancia(s) copiadas.")
                end},

                {"Cortar", function()
                    clipboardInstances = {}
                    for _, v in ipairs(selected) do
                        table.insert(clipboardInstances, v)
                    end
                    clipboardMode = "cut"
                    setStatus("Instancia(s) cortadas.")
                end},

                {"Colar como filho", function()
                    if #selected == 0 then return end
                    local target = selected[#selected]
                    local count = 0

                    for _, v in ipairs(clipboardInstances) do
                        if v and v.Parent and v ~= target
                        and not target:IsDescendantOf(v) then
                            pcall(function()
                                if clipboardMode == "copy" then
                                    local clone = v:Clone()
                                    clone.Parent = target
                                else
                                    v.Parent = target
                                end
                                count += 1
                            end)
                        end
                    end

                    setStatus("Colagem concluida: " .. count)
                    buildTree()
                end},

                {"Duplicar", function()
                    local count = 0
                    for _, v in ipairs(selected) do
                        pcall(function()
                            local clone = v:Clone()
                            clone.Parent = v.Parent
                            count += 1
                        end)
                    end
                    setStatus("Duplicadas: " .. count)
                    buildTree()
                end},

                {"Apagar", function()
                    local count = 0
                    for _, v in ipairs(selected) do
                        pcall(function()
                            if v ~= game and v.Parent then
                                v:Destroy()
                                count += 1
                            end
                        end)
                    end

                    setSelection({})
                    setStatus("Apagadas: " .. count)
                    buildTree()
                    showProperties()
                end},

                {"Renomear", function()
                    local target = selected[#selected]
                    if not target then return end

                    local rename = create("Frame", {
                        Size = UDim2.new(0, 300, 0, 112),
                        Position = UDim2.new(0.5, -150, 0.5, -56),
                        BackgroundColor3 = COLORS.header,
                        BorderSizePixel = 0,
                        ZIndex = 80,
                    }, gui)

                    corner(rename, 8)
                    stroke(rename)

                    label(rename, "Novo nome",
                        UDim2.new(1, -16, 0, 25),
                        UDim2.new(0, 8, 0, 4), 12)

                    local box = create("TextBox", {
                        Size = UDim2.new(1, -16, 0, 30),
                        Position = UDim2.new(0, 8, 0, 33),
                        BackgroundColor3 = COLORS.panel,
                        TextColor3 = COLORS.text,
                        Text = target.Name,
                        ClearTextOnFocus = false,
                        Font = Enum.Font.Code,
                        TextSize = 13,
                        ZIndex = 81,
                    }, rename)

                    corner(box, 5)

                    button(rename, "Salvar",
                        UDim2.new(0, 80, 0, 25),
                        UDim2.new(1, -88, 1, -32),
                        function()
                            local ok = pcall(function()
                                target.Name = box.Text
                            end)

                            setStatus(ok and "Renomeado."
                                or "Nao foi possivel renomear.")

                            rename:Destroy()
                            buildTree()
                        end,
                        COLORS.accent).ZIndex = 81
                end},

                {"Copiar caminho", function()
                    local path = ""
                    local current = selected[#selected]

                    while current and current ~= game do
                        path = current.Name
                            .. (path == "" and "" or "." .. path)
                        current = current.Parent
                    end

                    if requestClipboard then
                        pcall(requestClipboard, path)
                    end

                    setStatus("Caminho: " .. path)
                end},

                {"Criar objeto...", function()
                    local target = selected[#selected]
                    if not target then return end

                    local createFrame = create("Frame", {
                        Size = UDim2.new(0, 300, 0, 160),
                        Position = UDim2.new(0.5, -150, 0.5, -80),
                        BackgroundColor3 = COLORS.header,
                        BorderSizePixel = 0,
                        ZIndex = 80,
                    }, gui)

                    corner(createFrame, 8)
                    stroke(createFrame)

                    label(createFrame, "Criar instancia filha",
                        UDim2.new(1, -16, 0, 25),
                        UDim2.new(0, 8, 0, 4), 12)

                    local choices = {
                        "Folder", "Part", "Model", "StringValue",
                        "BoolValue", "IntValue", "NumberValue",
                        "Decal", "Attachment", "Highlight"
                    }

                    local selectedClass = "Folder"

                    local drop = create("TextButton", {
                        Size = UDim2.new(1, -16, 0, 30),
                        Position = UDim2.new(0, 8, 0, 34),
                        BackgroundColor3 = COLORS.panel,
                        Text = "Folder  ▼",
                        TextColor3 = COLORS.text,
                        TextSize = 12,
                        Font = Enum.Font.Code,
                        ZIndex = 81,
                    }, createFrame)

                    corner(drop, 5)

                    drop.MouseButton1Click:Connect(function()
                        local idx = table.find(choices, selectedClass) or 1
                        idx = idx % #choices + 1
                        selectedClass = choices[idx]
                        drop.Text = selectedClass .. "  ▼"
                    end)

                    button(createFrame, "Criar",
                        UDim2.new(0, 80, 0, 26),
                        UDim2.new(1, -88, 1, -34),
                        function()
                            local ok, err = pcall(function()
                                local obj = Instance.new(selectedClass)
                                obj.Name = selectedClass
                                obj.Parent = target

                                if obj:IsA("BasePart") then
                                    obj.Anchored = true
                                end
                            end)

                            setStatus(ok and ("Criado: " .. selectedClass)
                                or tostring(err))

                            createFrame:Destroy()
                            buildTree()
                        end,
                        COLORS.green).ZIndex = 81
                end},

                {"Cancelar", function() end},
            }

            for i, action in ipairs(actions) do
                local b = button(menu, action[1],
                    UDim2.new(1, -8, 0, 25),
                    UDim2.new(0, 4, 0, (i-1)*29+4),
                    function()
                        menu:Destroy()
                        pcall(action[2])
                    end)

                b.ZIndex = 51
            end
        end)

        if expanded[obj] or query ~= "" then
            local ok, children = pcall(function()
                return obj:GetChildren()
            end)

            if ok then
                table.sort(children, function(a, b)
                    return a.Name:lower() < b.Name:lower()
                end)

                for _, child in ipairs(children) do
                    draw(child, depth + 1)
                end
            end
        end
    end

    for _, root in ipairs(roots) do
        if root and root.Parent ~= nil then
            draw(root, 0)
        end
    end
end

refreshButton.MouseButton1Click:Connect(function()
    buildTree()
    showProperties()
    setStatus("Explorer atualizado.")
end)

expandButton.MouseButton1Click:Connect(function()
    for _, obj in ipairs(visibleInstances) do
        expanded[obj] = true
    end
    buildTree()
end)

collapseButton.MouseButton1Click:Connect(function()
    expanded = setmetatable({}, {__mode = "k"})
    buildTree()
end)

searchBox:GetPropertyChangedSignal("Text"):Connect(function()
    buildTree()
end)

-- Atalhos de teclado
UIS.InputBegan:Connect(function(input, processed)
    if processed or not gui.Parent then return end

    local ctrl = UIS:IsKeyDown(Enum.KeyCode.LeftControl)
        or UIS:IsKeyDown(Enum.KeyCode.RightControl)

    if ctrl and input.KeyCode == Enum.KeyCode.C then
        clipboardInstances = {}
        for _, obj in ipairs(selected) do
            table.insert(clipboardInstances, obj)
        end
        clipboardMode = "copy"
        setStatus("Copiado para clipboard interno.")

    elseif ctrl and input.KeyCode == Enum.KeyCode.X then
        clipboardInstances = {}
        for _, obj in ipairs(selected) do
            table.insert(clipboardInstances, obj)
        end
        clipboardMode = "cut"
        setStatus("Cortado para clipboard interno.")

    elseif ctrl and input.KeyCode == Enum.KeyCode.V then
        if #selected > 0 then
            local target = selected[#selected]
            local count = 0

            for _, obj in ipairs(clipboardInstances) do
                if obj and obj.Parent and obj ~= target
                and not target:IsDescendantOf(obj) then
                    pcall(function()
                        if clipboardMode == "copy" then
                            local clone = obj:Clone()
                            clone.Parent = target
                        else
                            obj.Parent = target
                        end
                        count += 1
                    end)
                end
            end

            setStatus("Colados " .. count .. " objeto(s).")
            buildTree()
        end

    elseif input.KeyCode == Enum.KeyCode.Delete
    or input.KeyCode == Enum.KeyCode.Backspace then
        local count = 0

        for _, obj in ipairs(selected) do
            pcall(function()
                if obj.Parent and obj ~= game then
                    obj:Destroy()
                    count += 1
                end
            end)
        end

        setSelection({})
        setStatus("Apagados " .. count .. " objeto(s).")
        buildTree()
        showProperties()

    elseif ctrl and input.KeyCode == Enum.KeyCode.D then
        local count = 0

        for _, obj in ipairs(selected) do
            pcall(function()
                local clone = obj:Clone()
                clone.Parent = obj.Parent
                count += 1
            end)
        end

        setStatus("Duplicados " .. count .. " objeto(s).")
        buildTree()

    elseif ctrl and input.KeyCode == Enum.KeyCode.A then
        setSelection(visibleInstances)
        buildTree()
        showProperties()
    end
end)

-- Atualiza quando instancias mudam no cliente
local updateQueued = false

local function queueUpdate()
    if updateQueued then return end
    updateQueued = true

    task.delay(1, function()
        updateQueued = false
        if gui.Parent then
            buildTree()
        end
    end)
end

for _, root in ipairs(getRoots()) do
    pcall(function()
        root.DescendantAdded:Connect(queueUpdate)
        root.DescendantRemoving:Connect(queueUpdate)
        root.ChildAdded:Connect(queueUpdate)
        root.ChildRemoved:Connect(queueUpdate)
    end)
end

buildTree()
showProperties()
setStatus("Pronto. Ctrl+clique ou Shift+clique para selecionar multiplos.")

-- DELTA EXPLORER - PARTE 3
-- Painel Properties avançado
-- Substitua a função showProperties() antiga por esta versão.
-- Mantenha as funções create, corner, label, safeGet,
-- valueString, setStatus e as variáveis globais da Parte 1.

local propertySearch = nil

local PROPERTY_GROUPS = {
    Identity = {
        "Name", "ClassName", "Archivable"
    },
    Transform = {
        "Position", "Orientation", "CFrame",
        "PivotOffset", "Size"
    },
    Appearance = {
        "Color", "BrickColor", "Material",
        "Transparency", "Reflectance", "CastShadow",
        "LocalTransparencyModifier"
    },
    Physics = {
        "Anchored", "CanCollide", "CanTouch",
        "CanQuery", "Massless", "AssemblyLinearVelocity",
        "AssemblyAngularVelocity"
    },
    GUI = {
        "Visible", "Enabled", "Text", "TextColor3",
        "TextTransparency", "BackgroundColor3",
        "BackgroundTransparency", "Image", "ImageColor3",
        "ImageTransparency", "TextSize", "Font"
    },
    Values = {
        "Value", "MinValue", "MaxValue"
    },
    Sound = {
        "Volume", "PlaybackSpeed", "Looped", "Playing"
    },
    Humanoid = {
        "WalkSpeed", "JumpPower", "JumpHeight",
        "Health", "MaxHealth", "HipHeight"
    },
    Other = {
        "DisplayName", "Texture", "TextureID",
        "MeshId", "DoubleSided"
    }
}

local READ_ONLY_PROPERTIES = {
    ClassName = true,
    Parent = true,
    CFrame = true,
    Source = true,
    MeshId = true,
    TextureID = true,
    AssemblyLinearVelocity = true,
    AssemblyAngularVelocity = true,
}

local function parseVector(text, dimensions)
    local values = {}
    for part in string.gmatch(text, "[^,]+") do
        local number = tonumber(part:match("^%s*(.-)%s*$"))
        if number == nil then
            return nil
        end
        table.insert(values, number)
    end

    if #values ~= dimensions then
        return nil
    end

    return values
end

local function convertPropertyValue(prop, oldValue, text)
    local kind = typeof(oldValue)

    if READ_ONLY_PROPERTIES[prop] then
        return nil, "Propriedade somente leitura neste painel."
    end

    if kind == "string" then
        return text
    elseif kind == "boolean" then
        local lower = string.lower(text)
        if lower == "true" then return true end
        if lower == "false" then return false end
        return nil, "Digite true ou false."
    elseif kind == "number" then
        local number = tonumber(text)
        if number == nil then
            return nil, "Número inválido."
        end
        return number
    elseif kind == "Color3" then
        local values = parseVector(text, 3)
        if not values then
            return nil, "Use R, G, B."
        end

        local r, g, b = values[1], values[2], values[3]
        if math.max(r, g, b) > 1 then
            r, g, b = r / 255, g / 255, b / 255
        end

        return Color3.new(
            math.clamp(r, 0, 1),
            math.clamp(g, 0, 1),
            math.clamp(b, 0, 1)
        )
    elseif kind == "Vector3" then
        local v = parseVector(text, 3)
        if not v then return nil, "Use X, Y, Z." end
        return Vector3.new(v[1], v[2], v[3])
    elseif kind == "Vector2" then
        local v = parseVector(text, 2)
        if not v then return nil, "Use X, Y." end
        return Vector2.new(v[1], v[2])
    elseif kind == "UDim2" then
        local v = parseVector(text, 4)
        if not v then
            return nil, "Use escalaX, offsetX, escalaY, offsetY."
        end
        return UDim2.new(v[1], v[2], v[3], v[4])
    elseif kind == "EnumItem" then
        local enumType = tostring(oldValue):match("^Enum%.([^%.]+)%.")
        if not enumType then
            return nil, "Não foi possível identificar o Enum."
        end

        local enumName = text:match("([^%.]+)$") or text
        local ok, result = pcall(function()
            return Enum[enumType][enumName]
        end)

        if ok and result then
            return result
        end

        return nil, "Valor de Enum inválido."
    end

    return nil, "Tipo não editável: " .. kind
end

local function propertyCategory(prop)
    for category, names in pairs(PROPERTY_GROUPS) do
        for _, name in ipairs(names) do
            if name == prop then
                return category
            end
        end
    end
    return "Other"
end

local function showProperties()
    propertyRefreshToken += 1
    local token = propertyRefreshToken

    clearGuiChildren(propsScroll)

    if #selected == 0 then
        label(propsScroll, "Nenhuma instância selecionada.",
            UDim2.new(1, -8, 0, 30),
            UDim2.new(), 12, COLORS.muted)
        return
    end

    local obj = selected[#selected]

    label(propsScroll, obj.Name,
        UDim2.new(1, -8, 0, 24),
        UDim2.new(), 13, COLORS.text)

    label(propsScroll,
        obj.ClassName .. " • " .. tostring(#selected) .. " selecionada(s)",
        UDim2.new(1, -8, 0, 22),
        UDim2.new(), 11, COLORS.muted)

    if not propertySearch or not propertySearch.Parent then
        propertySearch = create("TextBox", {
            Name = "PropertySearch",
            Size = UDim2.new(1, -8, 0, 30),
            BackgroundColor3 = COLORS.header,
            TextColor3 = COLORS.text,
            PlaceholderColor3 = COLORS.muted,
            PlaceholderText = "Pesquisar propriedade...",
            Text = "",
            ClearTextOnFocus = false,
            TextSize = 12,
            Font = Enum.Font.Code,
            BorderSizePixel = 0,
            LayoutOrder = 3,
        }, propsScroll)
        corner(propertySearch, 5)
    end

    local query = string.lower(propertySearch.Text or "")
    local properties = {}

    for _, names in pairs(PROPERTY_GROUPS) do
        for _, prop in ipairs(names) do
            if not table.find(properties, prop) then
                table.insert(properties, prop)
            end
        end
    end

    -- Também inclui propriedades adicionais que sejam comuns.
    for _, prop in ipairs({
        "Name", "Archivable", "Anchored", "CanCollide",
        "CanTouch", "CanQuery", "Transparency", "Color",
        "Material", "Size", "Position", "Orientation",
        "Visible", "Enabled", "Text", "TextColor3",
        "BackgroundColor3", "Value", "WalkSpeed",
        "JumpPower", "Health", "MaxHealth"
    }) do
        if not table.find(properties, prop) then
            table.insert(properties, prop)
        end
    end

    table.sort(properties, function(a, b)
        return a:lower() < b:lower()
    end)

    local order = 10
    local lastCategory = ""

    for _, prop in ipairs(properties) do
        if query == "" or string.find(
            string.lower(prop), query, 1, true
        ) then
            local ok, currentValue = safeGet(obj, prop)

            if ok then
                local category = propertyCategory(prop)

                if category ~= lastCategory then
                    order += 1
                    local header = label(
                        propsScroll,
                        string.upper(category),
                        UDim2.new(1, -8, 0, 24),
                        UDim2.new(),
                        11,
                        COLORS.accent
                    )
                    header.LayoutOrder = order
                    lastCategory = category
                end

                order += 1

                local row = create("Frame", {
                    Size = UDim2.new(1, -2, 0, 44),
                    BackgroundColor3 =
                        order % 2 == 0 and COLORS.rowAlt or COLORS.row,
                    BorderSizePixel = 0,
                    LayoutOrder = order,
                }, propsScroll)
                corner(row, 4)

                label(row, prop,
                    UDim2.new(0.40, -5, 1, 0),
                    UDim2.new(0, 6, 0, 0),
                    10, COLORS.muted)

                local edit = create("TextBox", {
                    Size = UDim2.new(0.60, -10, 0, 27),
                    Position = UDim2.new(0.40, 0, 0, 3),
                    BackgroundColor3 = COLORS.header,
                    TextColor3 = COLORS.text,
                    PlaceholderColor3 = COLORS.muted,
                    Text = valueString(currentValue),
                    ClearTextOnFocus = false,
                    TextSize = 10,
                    Font = Enum.Font.Code,
                    TextXAlignment = Enum.TextXAlignment.Left,
                    BorderSizePixel = 0,
                }, row)
                corner(edit, 4)

                local restore = create("TextButton", {
                    Size = UDim2.new(0, 24, 0, 12),
                    Position = UDim2.new(1, -29, 1, -13),
                    BackgroundColor3 = COLORS.header,
                    Text = "↶",
                    TextColor3 = COLORS.muted,
                    TextSize = 10,
                    BorderSizePixel = 0,
                }, row)
                restore.ZIndex = 3

                restore.MouseButton1Click:Connect(function()
                    if token ~= propertyRefreshToken then return end
                    local nowOk, nowValue = safeGet(obj, prop)
                    if nowOk then
                        edit.Text = valueString(nowValue)
                    end
                end)

                edit.FocusLost:Connect(function(enterPressed)
                    if not enterPressed
                    or token ~= propertyRefreshToken then
                        return
                    end

                    if READ_ONLY_PROPERTIES[prop] then
                        setStatus(prop .. " é somente leitura neste painel.")
                        return
                    end

                    local okNow, oldValue = safeGet(obj, prop)
                    if not okNow then
                        setStatus("Não foi possível ler " .. prop .. ".")
                        return
                    end

                    local newValue, conversionError =
                        convertPropertyValue(prop, oldValue, edit.Text)

                    if newValue == nil then
                        setStatus(conversionError or "Valor inválido.")
                        edit.Text = valueString(oldValue)
                        return
                    end

                    local success, err = pcall(function()
                        for _, target in ipairs(selected) do
                            target[prop] = newValue
                        end
                    end)

                    if success then
                        setStatus("Atualizado: " .. prop)
                        showProperties()
                    else
                        setStatus("Não foi possível alterar " ..
                            prop .. ": " .. tostring(err))
                        edit.Text = valueString(oldValue)
                    end
                end)
            end
        end
    end
end

-- Atualiza o painel quando o texto de pesquisa mudar.
if propertySearch then
    propertySearch:GetPropertyChangedSignal("Text"):Connect(function()
        showProperties()
    end)
end

--==================================================
-- PART 4 - BOTAO SET
--==================================================

local setButton = Instance.new("TextButton")
setButton.Name = "SetButton"
setButton.Size = UDim2.new(0, 70, 0, 30)
setButton.Position = UDim2.new(1, -80, 0, 5)
setButton.BackgroundColor3 = Color3.fromRGB(50, 155, 90)
setButton.TextColor3 = Color3.new(1, 1, 1)
setButton.Text = "SET"
setButton.TextSize = 14
setButton.Font = Enum.Font.GothamBold
setButton.BorderSizePixel = 0
setButton.ZIndex = 20
setButton.Parent = propsFrame

local setCorner = Instance.new("UICorner")
setCorner.CornerRadius = UDim.new(0, 6)
setCorner.Parent = setButton

setButton.MouseButton1Click:Connect(function()
    local selectedObject = selected and selected[#selected]

    if not selectedObject then
        setButton.Text = "SELECIONE"
        task.delay(1.5, function()
            if setButton and setButton.Parent then
                setButton.Text = "SET"
            end
        end)
        return
    end

    local valueBox
    local propertyName

    for _, item in ipairs(propsScroll:GetDescendants()) do
        if item:IsA("TextBox") then
            local name = item:GetAttribute("PropertyName")

            if name and item:IsFocused() then
                valueBox = item
                propertyName = name
                break
            end
        end
    end

    if not valueBox or not propertyName then
        setButton.Text = "CLIQUE NO VALOR"
        task.delay(1.5, function()
            if setButton and setButton.Parent then
                setButton.Text = "SET"
            end
        end)
        return
    end

    local success, err = pcall(function()
        local oldValue = selectedObject[propertyName]
        local newValue = convertPropertyValue(
            propertyName,
            oldValue,
            valueBox.Text
        )

        selectedObject[propertyName] = newValue
    end)

    if success then
        setButton.Text = "APLICADO!"
        pcall(showProperties)
    else
        setButton.Text = "FALHOU"
        warn("[SET] Erro ao aplicar propriedade:", err)
    end

    task.delay(1.5, function()
        if setButton and setButton.Parent then
            setButton.Text = "SET"
        end
    end)
end)
