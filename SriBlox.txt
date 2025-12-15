-- Minimal ScriptBlox Search UI
-- filepath: SimpleScriptSearch.lua

local UIS = game:GetService("UserInputService")
local HttpService = game:GetService("HttpService")
local Players = game:GetService("Players")
local TweenService = game:GetService("TweenService")
local player = Players.LocalPlayer

-- Charger Neon (glow/blur Sirius)
local function requireFromUrl(url)
    local s, r = pcall(function()
        return loadstring(game:HttpGet(url))()
    end)
    if s then return r end
end
local Neon = requireFromUrl("https://raw.githubusercontent.com/shlexware/Sirius/request/library/neon.lua")

-- THEMES
local Themes = {
    ["Sombre"] = {
        Background = Color3.fromRGB(18, 18, 28),
        Card = Color3.fromRGB(24, 26, 38),
        Stroke = Color3.fromRGB(80, 120, 255),
        Text = Color3.fromRGB(230, 230, 255),
        Accent = Color3.fromRGB(80, 120, 255),
        RunBtn = Color3.fromRGB(60, 200, 120),
        RunBtnHover = Color3.fromRGB(80, 255, 140),
        LinkBtn = Color3.fromRGB(24, 26, 38),
        LinkBtnHover = Color3.fromRGB(80, 120, 255),
    },
    ["Clair"] = {
        Background = Color3.fromRGB(240, 240, 255),
        Card = Color3.fromRGB(220, 225, 255),
        Stroke = Color3.fromRGB(120, 160, 255),
        Text = Color3.fromRGB(40, 40, 60),
        Accent = Color3.fromRGB(120, 160, 255),
        RunBtn = Color3.fromRGB(60, 200, 120),
        RunBtnHover = Color3.fromRGB(80, 255, 140),
        LinkBtn = Color3.fromRGB(220, 225, 255),
        LinkBtnHover = Color3.fromRGB(120, 160, 255),
    },
    ["Colore"] = { -- Corrigé: plus d'accent
        Background = Color3.fromRGB(60, 40, 90),
        Card = Color3.fromRGB(80, 60, 120),
        Stroke = Color3.fromRGB(140, 100, 255),
        Text = Color3.fromRGB(220, 200, 255),
        Accent = Color3.fromRGB(140, 100, 255),
        RunBtn = Color3.fromRGB(120, 60, 255),
        RunBtnHover = Color3.fromRGB(180, 120, 255),
        LinkBtn = Color3.fromRGB(80, 60, 120),
        LinkBtnHover = Color3.fromRGB(140, 100, 255),
    }
}
local CurrentTheme = Themes["Sombre"]

-- UI Setup (modern, 2 columns)
local gui = Instance.new("ScreenGui")
gui.Name = "SribloxSearchUI"
gui.Parent = game:GetService("CoreGui")

-- Notification au chargement
pcall(function()
    game:GetService("StarterGui"):SetCore("SendNotification", {
        Title = "Sriblox chargé !",
        Text = "Appuie sur F6 pour ouvrir la recherche ScriptBlox.",
        Duration = 8
    })
end)

local searchBarFrame = Instance.new("Frame")
searchBarFrame.Size = UDim2.new(0, 520, 0, 38)
searchBarFrame.Position = UDim2.new(0.5, -260, 0.10, 0)
searchBarFrame.BackgroundColor3 = CurrentTheme.Background
searchBarFrame.BackgroundTransparency = 0
searchBarFrame.BorderSizePixel = 0
searchBarFrame.Visible = false
searchBarFrame.Parent = gui
local searchBarCorner = Instance.new("UICorner", searchBarFrame)
searchBarCorner.CornerRadius = UDim.new(0, 12)
local searchBarStroke = Instance.new("UIStroke", searchBarFrame)
searchBarStroke.Color = CurrentTheme.Stroke
searchBarStroke.Thickness = 2
searchBarStroke.Transparency = 0.18

local searchIcon = Instance.new("ImageLabel")
searchIcon.Size = UDim2.new(0, 24, 0, 24)
searchIcon.Position = UDim2.new(0, 10, 0.5, -12)
searchIcon.BackgroundTransparency = 1
searchIcon.Image = "rbxassetid://7733960981"
searchIcon.ImageColor3 = CurrentTheme.Accent
searchIcon.Parent = searchBarFrame

local searchBox = Instance.new("TextBox")
searchBox.Size = UDim2.new(1, -90, 1, -12)
searchBox.Position = UDim2.new(0, 40, 0, 6)
searchBox.BackgroundColor3 = CurrentTheme.Background
searchBox.TextColor3 = CurrentTheme.Text
searchBox.PlaceholderText = "Rechercher sur ScriptBlox.com"
searchBox.Text = ""
searchBox.Font = Enum.Font.GothamSemibold
searchBox.TextSize = 17
searchBox.ClearTextOnFocus = false
searchBox.BorderSizePixel = 0
searchBox.Parent = searchBarFrame
local searchBoxCorner = Instance.new("UICorner", searchBox)
searchBoxCorner.CornerRadius = UDim.new(0, 8)

-- Paramètres bouton
local settingsBtn = Instance.new("TextButton")
settingsBtn.Size = UDim2.new(0, 32, 0, 32)
settingsBtn.Position = UDim2.new(1, -38, 0.5, -16)
settingsBtn.BackgroundColor3 = CurrentTheme.LinkBtn
settingsBtn.Text = "⚙️"
settingsBtn.TextColor3 = CurrentTheme.Accent
settingsBtn.Font = Enum.Font.GothamBold
settingsBtn.TextSize = 18
settingsBtn.AutoButtonColor = true
settingsBtn.Parent = searchBarFrame
local settingsCorner = Instance.new("UICorner", settingsBtn)
settingsCorner.CornerRadius = UDim.new(0, 8)

-- Menu paramètres (popup)
local settingsMenu = Instance.new("Frame")
settingsMenu.Size = UDim2.new(0, 180, 0, 0) -- Hauteur auto
settingsMenu.Position = UDim2.new(1, -180, 1, 8)
settingsMenu.BackgroundColor3 = CurrentTheme.Card
settingsMenu.Visible = false
settingsMenu.Parent = searchBarFrame
local settingsMenuCorner = Instance.new("UICorner", settingsMenu)
settingsMenuCorner.CornerRadius = UDim.new(0, 10)
local settingsMenuStroke = Instance.new("UIStroke", settingsMenu)
settingsMenuStroke.Color = CurrentTheme.Stroke
settingsMenuStroke.Thickness = 1.2
settingsMenuStroke.Transparency = 0.25

-- Ajout d'un UIListLayout pour placer les boutons automatiquement
local themeListLayout = Instance.new("UIListLayout", settingsMenu)
themeListLayout.SortOrder = Enum.SortOrder.LayoutOrder
themeListLayout.Padding = UDim.new(0, 6)
themeListLayout.HorizontalAlignment = Enum.HorizontalAlignment.Center

-- Adapter dynamiquement la hauteur du menu aux enfants
settingsMenu.AutomaticSize = Enum.AutomaticSize.Y
settingsMenu.ClipsDescendants = false

local themeLabel = Instance.new("TextLabel")
themeLabel.Size = UDim2.new(1, 0, 0, 24)
themeLabel.Position = UDim2.new(0, 0, 0, 6)
themeLabel.BackgroundTransparency = 1
themeLabel.Text = "Thème :"
themeLabel.TextColor3 = CurrentTheme.Text
themeLabel.Font = Enum.Font.GothamBold
themeLabel.TextSize = 15
themeLabel.Parent = settingsMenu

local function createThemeBtn(txt, theme)
    local btn = Instance.new("TextButton")
    btn.Size = UDim2.new(1, -32, 0, 28) -- marge augmentée, hauteur +2px
    btn.BackgroundColor3 = theme.Card
    btn.Text = txt
    btn.TextColor3 = theme.Text
    btn.Font = Enum.Font.Gotham
    btn.TextSize = 15
    btn.AutoButtonColor = true
    btn.Parent = settingsMenu
    local btnCorner = Instance.new("UICorner", btn)
    btnCorner.CornerRadius = UDim.new(0, 8)
    btn.LayoutOrder = 10
    btn.MouseButton1Click:Connect(function()
        CurrentTheme = theme
        -- reload theme
        searchBarFrame.BackgroundColor3 = theme.Background
        searchBarStroke.Color = theme.Stroke
        searchBox.BackgroundColor3 = theme.Background
        searchBox.TextColor3 = theme.Text
        searchIcon.ImageColor3 = theme.Accent
        settingsBtn.BackgroundColor3 = theme.LinkBtn
        settingsBtn.TextColor3 = theme.Accent
        settingsMenu.BackgroundColor3 = theme.Card
        settingsMenuStroke.Color = theme.Stroke
        themeLabel.TextColor3 = theme.Text
        for _, c in ipairs(resultsFrame:GetChildren()) do
            if c:IsA("Frame") then
                c.BackgroundColor3 = theme.Card
                local s = c:FindFirstChildOfClass("UIStroke")
                if s then s.Color = theme.Stroke end
            end
        end
    end)
end
createThemeBtn("Sombre", Themes.Sombre)
createThemeBtn("Clair", Themes.Clair)
createThemeBtn("Coloré", Themes.Colore)

settingsBtn.MouseButton1Click:Connect(function()
    settingsMenu.Visible = not settingsMenu.Visible
end)

-- Results list (2 columns, plus grand)
local resultsFrame = Instance.new("ScrollingFrame")
resultsFrame.Size = UDim2.new(0, 540, 0, 560)
resultsFrame.Position = UDim2.new(0.5, -270, 0.10, 48)
resultsFrame.BackgroundColor3 = CurrentTheme.Background
resultsFrame.BackgroundTransparency = 0
resultsFrame.BorderSizePixel = 0
resultsFrame.Visible = false
resultsFrame.Parent = gui
resultsFrame.CanvasSize = UDim2.new(0,0,0,0)
resultsFrame.ScrollBarThickness = 7
local resultsCorner = Instance.new("UICorner", resultsFrame)
resultsCorner.CornerRadius = UDim.new(0, 12)
local resultsStroke = Instance.new("UIStroke", resultsFrame)
resultsStroke.Color = CurrentTheme.Stroke
resultsStroke.Thickness = 1.5
resultsStroke.Transparency = 0.22

-- 2 colonnes layout
local UIGrid = Instance.new("UIGridLayout", resultsFrame)
UIGrid.CellSize = UDim2.new(0, 250, 0, 110)
UIGrid.CellPadding = UDim2.new(0, 16, 0, 16)
UIGrid.SortOrder = Enum.SortOrder.LayoutOrder
UIGrid.FillDirectionMaxCells = 2
UIGrid.FillDirection = Enum.FillDirection.Horizontal
UIGrid.HorizontalAlignment = Enum.HorizontalAlignment.Center
UIGrid.VerticalAlignment = Enum.VerticalAlignment.Top

-- Helper: format numbers (e.g. 1200 -> 1.2K)
local function formatNumber(n)
    if not n then return "0" end
    if n >= 1e6 then return string.format("%.1fM", n/1e6) end
    if n >= 1e3 then return string.format("%.1fK", n/1e3) end
    return tostring(n)
end

-- Helper: format date
local function formatDate(str)
    if not str then return "" end
    local y, m, d = str:match("(%d+)%-(%d+)%-(%d+)")
    if y and m and d then return d.."/"..m.."/"..y end
    return str
end

-- Loader animé pendant la recherche
local loader = Instance.new("TextLabel")
loader.Size = UDim2.new(0, 120, 0, 32)
loader.Position = UDim2.new(0.5, -60, 0.18, 60)
loader.BackgroundTransparency = 1
loader.Text = "Chargement..."
loader.TextColor3 = Color3.fromRGB(60, 120, 255)
loader.Font = Enum.Font.GothamBold
loader.TextSize = 18
loader.Visible = false
loader.Parent = gui

-- Message Aucun résultat
local noResult = Instance.new("TextLabel")
noResult.Size = UDim2.new(0, 320, 0, 32)
noResult.Position = UDim2.new(0.5, -160, 0.18, 60)
noResult.BackgroundTransparency = 1
noResult.Text = "Aucun résultat."
noResult.TextColor3 = Color3.fromRGB(180, 180, 180)
noResult.Font = Enum.Font.GothamBold
noResult.TextSize = 18
noResult.Visible = false
noResult.Parent = gui

-- Show/hide UI with F6
UIS.InputBegan:Connect(function(input, processed)
    if processed then return end
    if input.KeyCode == Enum.KeyCode.F6 then
        searchBarFrame.Visible = not searchBarFrame.Visible
        resultsFrame.Visible = false
        if searchBarFrame.Visible then
            TweenService:Create(searchBarFrame, TweenInfo.new(0.35, Enum.EasingStyle.Quint), {BackgroundTransparency = 0.15}):Play()
            searchBox:CaptureFocus()
        else
            TweenService:Create(searchBarFrame, TweenInfo.new(0.25, Enum.EasingStyle.Quint), {BackgroundTransparency = 1}):Play()
            TweenService:Create(resultsFrame, TweenInfo.new(0.25, Enum.EasingStyle.Quint), {BackgroundTransparency = 1}):Play()
        end
    end
end)

-- Sirius-style search bar color feedback
searchBox:GetPropertyChangedSignal("Text"):Connect(function()
    if #searchBox.Text > 0 then
        TweenService:Create(searchIcon, TweenInfo.new(.5,Enum.EasingStyle.Quint),  {ImageColor3 = Color3.fromRGB(80, 120, 255)}):Play()
        TweenService:Create(searchBarStroke, TweenInfo.new(.5,Enum.EasingStyle.Quint),  {Color = Color3.fromRGB(80, 120, 255)}):Play()
        TweenService:Create(searchBox, TweenInfo.new(.5,Enum.EasingStyle.Quint),  {TextColor3 = Color3.fromRGB(230, 230, 255)}):Play()
    else
        TweenService:Create(searchIcon, TweenInfo.new(.5,Enum.EasingStyle.Quint),  {ImageColor3 = Color3.fromRGB(120, 140, 255)}):Play()
        TweenService:Create(searchBarStroke, TweenInfo.new(.5,Enum.EasingStyle.Quint),  {Color = Color3.fromRGB(80, 120, 255)}):Play()
        TweenService:Create(searchBox, TweenInfo.new(.5,Enum.EasingStyle.Quint),  {TextColor3 = Color3.fromRGB(230, 230, 255)}):Play()
    end
end)

-- Helper: create a modern script card (2 columns, new style)
local function createScriptCard(script)
    local card = Instance.new("Frame")
    card.Size = UDim2.new(0, 240, 0, 100)
    card.BackgroundColor3 = CurrentTheme.Card
    card.BackgroundTransparency = 0
    card.BorderSizePixel = 0
    card.Parent = resultsFrame
    local cardCorner = Instance.new("UICorner", card)
    cardCorner.CornerRadius = UDim.new(0, 10)
    local cardStroke = Instance.new("UIStroke", card)
    cardStroke.Color = CurrentTheme.Stroke
    cardStroke.Thickness = 1.2
    cardStroke.Transparency = 0.18

    -- Title
    local title = Instance.new("TextLabel")
    title.Size = UDim2.new(1, -16, 0, 22)
    title.Position = UDim2.new(0, 8, 0, 6)
    title.BackgroundTransparency = 1
    title.Text = script.title or "Unknown Title"
    title.TextColor3 = CurrentTheme.Accent
    title.Font = Enum.Font.GothamBold
    title.TextSize = 16
    title.TextXAlignment = Enum.TextXAlignment.Left
    title.Parent = card

    -- Tags container
    local tags = Instance.new("Frame")
    tags.Size = UDim2.new(1, -16, 0, 16)
    tags.Position = UDim2.new(0, 8, 0, 28)
    tags.BackgroundTransparency = 1
    tags.Parent = card
    local tagsLayout = Instance.new("UIListLayout", tags)
    tagsLayout.FillDirection = Enum.FillDirection.Horizontal
    tagsLayout.Padding = UDim.new(0, 4)
    tagsLayout.SortOrder = Enum.SortOrder.LayoutOrder

    local function addTag(text, color)
        local tag = Instance.new("TextLabel")
        tag.Size = UDim2.new(0, 54, 1, 0)
        tag.BackgroundColor3 = color
        tag.BackgroundTransparency = 0.08
        tag.Text = text
        tag.TextColor3 = Color3.fromRGB(255,255,255)
        tag.Font = Enum.Font.GothamBold
        tag.TextSize = 12
        tag.Parent = tags
        tag.TextXAlignment = Enum.TextXAlignment.Center
        tag.TextYAlignment = Enum.TextYAlignment.Center
        local tagCorner = Instance.new("UICorner", tag)
        tagCorner.CornerRadius = UDim.new(0, 6)
        return tag
    end
    if script.tags then
        for _, tag in ipairs(script.tags) do
            if tag == "REVIEW" then addTag("REVIEW", Color3.fromRGB(255, 196, 0)) end
            if tag == "VERIFIED" then addTag("VERIFIED", Color3.fromRGB(60, 200, 120)) end
            if tag == "CREATOR" then addTag("CREATOR", Color3.fromRGB(80, 120, 255)) end
            if tag == "UNIVERSAL" then addTag("UNIVERSAL", Color3.fromRGB(120, 60, 255)) end
            if tag == "KEY" then addTag("KEY", Color3.fromRGB(255, 60, 120)) end
            if tag == "PATCHED" then addTag("PATCHED", Color3.fromRGB(200, 60, 60)) end
            if tag == "TRENDING" then addTag("TRENDING", Color3.fromRGB(255, 120, 60)) end
        end
    end

    -- Author
    local author = Instance.new("TextLabel")
    author.Size = UDim2.new(0, 120, 0, 16)
    author.Position = UDim2.new(0, 8, 0, 48)
    author.BackgroundTransparency = 1
    author.Text = script.owner and ("by "..script.owner.username) or "by Unknown"
    author.TextColor3 = CurrentTheme.Text
    author.Font = Enum.Font.Gotham
    author.TextSize = 12
    author.TextXAlignment = Enum.TextXAlignment.Left
    author.Parent = card

    -- Description
    local desc = Instance.new("TextLabel")
    desc.Size = UDim2.new(1, -16, 0, 16)
    desc.Position = UDim2.new(0, 8, 0, 66)
    desc.BackgroundTransparency = 1
    desc.Text = script.features or script.game and script.game.name or "No description"
    desc.TextColor3 = CurrentTheme.Accent
    desc.Font = Enum.Font.Gotham
    desc.TextSize = 12
    desc.TextXAlignment = Enum.TextXAlignment.Left
    desc.TextYAlignment = Enum.TextYAlignment.Top
    desc.TextWrapped = true
    desc.Parent = card

    -- Stats badges
    local statsFrame = Instance.new("Frame")
    statsFrame.Size = UDim2.new(0, 120, 0, 16)
    statsFrame.Position = UDim2.new(1, -128, 0, 48)
    statsFrame.BackgroundTransparency = 1
    statsFrame.Parent = card
    local statsLayout = Instance.new("UIListLayout", statsFrame)
    statsLayout.FillDirection = Enum.FillDirection.Horizontal
    statsLayout.Padding = UDim.new(0, 3)
    statsLayout.SortOrder = Enum.SortOrder.LayoutOrder

    local function statBadge(txt, color)
        local badge = Instance.new("TextLabel")
        badge.Size = UDim2.new(0, 32, 1, 0)
        badge.BackgroundColor3 = color
        badge.BackgroundTransparency = 0.15
        badge.Text = txt
        badge.TextColor3 = Color3.fromRGB(255,255,255)
        badge.Font = Enum.Font.GothamBold
        badge.TextSize = 11
        badge.TextXAlignment = Enum.TextXAlignment.Center
        badge.TextYAlignment = Enum.TextYAlignment.Center
        badge.Parent = statsFrame
        local badgeCorner = Instance.new("UICorner", badge)
        badgeCorner.CornerRadius = UDim.new(0, 5)
    end
    statBadge("👁 "..formatNumber(script.views or 0), CurrentTheme.Accent)
    statBadge("👍 "..formatNumber(script.likes or 0), Color3.fromRGB(60, 200, 120))
    statBadge(script.type or "?", Color3.fromRGB(120, 60, 255))

    -- Run button (vert Roblox)
    local runBtn = Instance.new("TextButton")
    runBtn.Size = UDim2.new(0, 54, 0, 28)
    runBtn.Position = UDim2.new(1, -62, 1, -36)
    runBtn.BackgroundColor3 = CurrentTheme.RunBtn
    runBtn.Text = "▶"
    runBtn.TextColor3 = Color3.fromRGB(255,255,255)
    runBtn.Font = Enum.Font.GothamBold
    runBtn.TextSize = 18
    runBtn.AutoButtonColor = true
    runBtn.Parent = card
    local btnCorner = Instance.new("UICorner", runBtn)
    btnCorner.CornerRadius = UDim.new(0, 8)

    runBtn.MouseEnter:Connect(function()
        TweenService:Create(runBtn, TweenInfo.new(0.18, Enum.EasingStyle.Quint), {BackgroundColor3 = CurrentTheme.RunBtnHover}):Play()
    end)
    runBtn.MouseLeave:Connect(function()
        TweenService:Create(runBtn, TweenInfo.new(0.18, Enum.EasingStyle.Quint), {BackgroundColor3 = CurrentTheme.RunBtn}):Play()
    end)
    runBtn.MouseButton1Click:Connect(function()
        runBtn.Text = "⏳"
        local slug = script.slug
        local codeUrl = "https://www.scriptblox.com/api/script/"..slug
        local ok, codeResp = pcall(function()
            return HttpService:JSONDecode(game:HttpGet(codeUrl))
        end)
        if ok and codeResp and codeResp.script and codeResp.script.script then
            loadstring(codeResp.script.script)()
            runBtn.Text = "▶"
        else
            runBtn.Text = "❌"
            wait(1)
            runBtn.Text = "▶"
        end
    end)

    -- Lien ScriptBlox
    local linkBtn = Instance.new("TextButton")
    linkBtn.Size = UDim2.new(0, 24, 0, 24)
    linkBtn.Position = UDim2.new(1, -32, 0, 8)
    linkBtn.BackgroundColor3 = CurrentTheme.LinkBtn
    linkBtn.Text = "🔗"
    linkBtn.TextColor3 = CurrentTheme.Accent
    linkBtn.Font = Enum.Font.GothamBold
    linkBtn.TextSize = 14
    linkBtn.AutoButtonColor = true
    linkBtn.Parent = card
    local linkCorner = Instance.new("UICorner", linkBtn)
    linkCorner.CornerRadius = UDim.new(0, 6)
    linkBtn.MouseEnter:Connect(function()
        TweenService:Create(linkBtn, TweenInfo.new(0.15, Enum.EasingStyle.Quint), {BackgroundColor3 = CurrentTheme.LinkBtnHover}):Play()
    end)
    linkBtn.MouseLeave:Connect(function()
        TweenService:Create(linkBtn, TweenInfo.new(0.15, Enum.EasingStyle.Quint), {BackgroundColor3 = CurrentTheme.LinkBtn}):Play()
    end)
    linkBtn.MouseButton1Click:Connect(function()
        setclipboard("https://scriptblox.com/script/"..(script.slug or ""))
        linkBtn.Text = "✔"
        wait(0.7)
        linkBtn.Text = "🔗"
    end)

    -- Apparition animée
    card.BackgroundTransparency = 1
    card.Visible = false
    delay(0.03, function()
        card.Visible = true
        TweenService:Create(card, TweenInfo.new(0.18, Enum.EasingStyle.Quint), {BackgroundTransparency = 0}):Play()
    end)

    return card
end

-- Search on Enter (avec loader, no result, anims)
searchBox.FocusLost:Connect(function(enterPressed)
    if enterPressed and #searchBox.Text > 0 then
        for _, child in ipairs(resultsFrame:GetChildren()) do
            if child:IsA("Frame") then child:Destroy() end
        end
        resultsFrame.Visible = false
        noResult.Visible = false
        loader.Visible = true
        loader.Text = "Chargement..."
        local url = "https://scriptblox.com/api/script/search?q="..HttpService:UrlEncode(searchBox.Text).."&mode=free&max=12&page=1"
        local success, response = pcall(function()
            return HttpService:JSONDecode(game:HttpGet(url))
        end)
        loader.Visible = false
        if success and response and response.result and response.result.scripts then
            local scripts = response.result.scripts
            if #scripts == 0 then
                noResult.Visible = true
                resultsFrame.Visible = false
            else
                noResult.Visible = false
                resultsFrame.Visible = true
                local y = 0
                for _, script in ipairs(scripts) do
                    local card = createScriptCard(script)
                    card.LayoutOrder = y
                    y = y + 1
                end
                resultsFrame.CanvasSize = UDim2.new(0,0,0,math.max(0, y*100))
            end
        else
            noResult.Visible = true
            resultsFrame.Visible = false
        end
    else
        resultsFrame.Visible = false
        noResult.Visible = false
    end
end)

searchBox.Focused:Connect(function()
    if #searchBox.Text > 0 then
        TweenService:Create(searchIcon, TweenInfo.new(.5,Enum.EasingStyle.Quint),  {ImageColor3 = Color3.fromRGB(255, 255, 255)}):Play()
        TweenService:Create(searchBox, TweenInfo.new(.5,Enum.EasingStyle.Quint),  {TextColor3 = Color3.fromRGB(255, 255, 255)}):Play()
    end
end)