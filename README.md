```tsx
import { Box, Typography, Paper, List, ListItem, ListItemIcon, ListItemText, IconButton, Tooltip, Collapse } from '@mui/material';
import BarChartIcon from '@mui/icons-material/BarChart';
import ShowChartIcon from '@mui/icons-material/ShowChart';
import PieChartIcon from '@mui/icons-material/PieChart';
import BubbleChartIcon from '@mui/icons-material/BubbleChart';
import TimelineIcon from '@mui/icons-material/Timeline';
import ArrowForwardIosIcon from '@mui/icons-material/ArrowForwardIos';
import ArrowBackIosNewIcon from '@mui/icons-material/ArrowBackIosNew';
import { useState } from 'react';

// Define Widget interface representing each available widget
interface Widget {
  type: 'AreaChart' | 'ScatterPlot' | 'PieChart' | 'BubbleChart' | 'BarChart';
  label: string;
  icon: React.ReactNode;
  description: string;
}

interface SettingsProps {
  selectedWidgets: Set<Widget['type']>;
}

const Settings = ({ selectedWidgets }: SettingsProps) => {
  const [isExpanded, setIsExpanded] = useState(true);

  const handleDragStart = (e: React.DragEvent<HTMLDivElement>, widgetType: Widget['type']) => {
    if (selectedWidgets.has(widgetType)) {
      e.preventDefault();
      return;
    }
    e.dataTransfer.setData('widget', widgetType);
    e.dataTransfer.effectAllowed = 'move';
  };

  const widgets: Widget[] = [
    {
      type: 'AreaChart',
      label: 'Area Chart',
      icon: <TimelineIcon />,
      description: 'Visualize data trends over time with a filled area'
    },
    {
      type: 'ScatterPlot',
      label: 'Scatter Plot',
      icon: <ShowChartIcon />,
      description: 'Display relationships between two variables'
    },
    {
      type: 'PieChart',
      label: 'Pie Chart',
      icon: <PieChartIcon />,
      description: 'Show proportional data as slices of a circle'
    },
    {
      type: 'BubbleChart',
      label: 'Bubble Chart',
      icon: <BubbleChartIcon />,
      description: 'Represent three dimensions of data with size and position'
    },
    {
      type: 'BarChart',
      label: 'Bar Chart',
      icon: <BarChartIcon />,
      description: 'Compare values across categories'
    }
  ];

  return (
    <Box
      sx={{
        height: '100%',
        display: 'flex',
        flexDirection: 'row',
        position: 'relative',
        width: isExpanded ? 250 : 64,
      }}
    >
      {isExpanded ? (
        <Box
          sx={{
            width: '100%',
            height: '100%',
            display: 'flex',
            flexDirection: 'column',
            p: 2,
          }}
        >
          <Box
            sx={{
              display: 'flex',
              alignItems: 'center',
              mb: 2,
            }}
          >
            <IconButton
              onClick={() => setIsExpanded(false)}
              size="small"
              sx={{
                mr: 1,
                color: 'text.secondary',
                '&:hover': {
                  color: 'primary.main',
                },
              }}
            >
              <ArrowForwardIosIcon sx={{ fontSize: 18 }} />
            </IconButton>
            <Typography variant="h6">
              Available Widgets
            </Typography>
          </Box>

          <List
            sx={{
              flex: 1,
              overflowY: 'auto',
              '&::-webkit-scrollbar': {
                width: '8px',
              },
              '&::-webkit-scrollbar-track': {
                background: '#f1f1f1',
                borderRadius: '4px',
              },
              '&::-webkit-scrollbar-thumb': {
                background: '#888',
                borderRadius: '4px',
                '&:hover': {
                  background: '#666',
                },
              },
            }}
          >
            {widgets.map((widget) => {
              const isSelected = selectedWidgets.has(widget.type);
              return (
                <ListItem
                  key={widget.type}
                  draggable={!isSelected}
                  onDragStart={(e) => handleDragStart(e, widget.type)}
                  component={Paper}
                  sx={{
                    mb: 2,
                    cursor: isSelected ? 'not-allowed' : 'grab',
                    opacity: isSelected ? 0.5 : 1,
                    backgroundColor: isSelected ? 'action.disabledBackground' : 'background.paper',
                    '&:hover': !isSelected ? {
                      backgroundColor: 'action.hover',
                      transform: 'translateY(-2px)',
                    } : {},
                    transition: 'all 0.2s ease',
                    borderRadius: 1,
                  }}
                >
                  <ListItemIcon sx={{ color: isSelected ? 'action.disabled' : 'primary.main' }}>
                    {widget.icon}
                  </ListItemIcon>
                  <ListItemText
                    primary={widget.label}
                    secondary={isSelected ? 'Already added to dashboard' : widget.description}
                    primaryTypographyProps={{
                      fontWeight: 'medium',
                      color: isSelected ? 'text.disabled' : 'text.primary'
                    }}
                  />
                </ListItem>
              );
            })}
          </List>
        </Box>
      ) : (
        <Box
          sx={{
            width: '100%',
            height: '100%',
            display: 'flex',
            flexDirection: 'column',
            alignItems: 'center',
            backgroundColor: 'background.paper',
            pt: 2,
            pb: 2,
          }}
        >
          <IconButton
            onClick={() => setIsExpanded(true)}
            size="small"
            sx={{
              mb: 3,
              color: 'text.secondary',
              '&:hover': {
                color: 'primary.main',
              },
            }}
          >
            <ArrowBackIosNewIcon sx={{ fontSize: 20 }} />
          </IconButton>

          <Box
            sx={{
              display: 'flex',
              flexDirection: 'column',
              gap: 2,
              alignItems: 'center',
            }}
          >
            {widgets.map((widget) => {
              const isSelected = selectedWidgets.has(widget.type);
              return (
                <Tooltip key={widget.type} title={widget.label} placement="left">
                  <Paper
                    draggable={!isSelected}
                    onDragStart={(e) => handleDragStart(e, widget.type)}
                    sx={{
                      p: 1,
                      width: 40,
                      height: 40,
                      cursor: isSelected ? 'not-allowed' : 'grab',
                      opacity: isSelected ? 0.5 : 1,
                      backgroundColor: isSelected ? 'action.disabledBackground' : 'background.paper',
                      '&:hover': !isSelected ? {
                        backgroundColor: 'action.hover',
                        transform: 'translateX(-2px)',
                      } : {},
                      transition: 'all 0.2s ease',
                      display: 'flex',
                      alignItems: 'center',
                      justifyContent: 'center',
                    }}
                  >
                    <Box 
                      sx={{ 
                        color: isSelected ? 'action.disabled' : 'primary.main',
                        display: 'flex',
                        '& > svg': {
                          fontSize: 24,
                        }
                      }}
                    >
                      {widget.icon}
                    </Box>
                  </Paper>
                </Tooltip>
              );
            })}
          </Box>
        </Box>
      )}
    </Box>
  );
};

export default Settings; 
```



```tsx
import { Box, Typography, Paper, List, ListItem, ListItemIcon, ListItemText, IconButton, Tooltip, Collapse } from '@mui/material';
import BarChartIcon from '@mui/icons-material/BarChart';
import ShowChartIcon from '@mui/icons-material/ShowChart';
import PieChartIcon from '@mui/icons-material/PieChart';
import BubbleChartIcon from '@mui/icons-material/BubbleChart';
import TimelineIcon from '@mui/icons-material/Timeline';
import ArrowForwardIosIcon from '@mui/icons-material/ArrowForwardIos';
import ArrowBackIosNewIcon from '@mui/icons-material/ArrowBackIosNew';
import { useState } from 'react';

// Define Widget interface representing each available widget
interface Widget {
  type: 'AreaChart' | 'ScatterPlot' | 'PieChart' | 'BubbleChart' | 'BarChart';
  label: string;
  icon: React.ReactNode;
  description: string;
}

interface SettingsProps {
  selectedWidgets: Set<Widget['type']>;
}

const Settings = ({ selectedWidgets }: SettingsProps) => {
  const [isExpanded, setIsExpanded] = useState(true);

  const handleDragStart = (e: React.DragEvent<HTMLDivElement>, widgetType: Widget['type']) => {
    if (selectedWidgets.has(widgetType)) {
      e.preventDefault();
      return;
    }
    e.dataTransfer.setData('widget', widgetType);
    e.dataTransfer.effectAllowed = 'move';
  };

  const widgets: Widget[] = [
    {
      type: 'AreaChart',
      label: 'Area Chart',
      icon: <TimelineIcon />,
      description: 'Visualize data trends over time with a filled area'
    },
    {
      type: 'ScatterPlot',
      label: 'Scatter Plot',
      icon: <ShowChartIcon />,
      description: 'Display relationships between two variables'
    },
    {
      type: 'PieChart',
      label: 'Pie Chart',
      icon: <PieChartIcon />,
      description: 'Show proportional data as slices of a circle'
    },
    {
      type: 'BubbleChart',
      label: 'Bubble Chart',
      icon: <BubbleChartIcon />,
      description: 'Represent three dimensions of data with size and position'
    },
    {
      type: 'BarChart',
      label: 'Bar Chart',
      icon: <BarChartIcon />,
      description: 'Compare values across categories'
    }
  ];

  return (
    <Box
      sx={{
        height: '100%',
        display: 'flex',
        flexDirection: 'row',
        position: 'relative',
        width: isExpanded ? 250 : 64,
      }}
    >
      {isExpanded ? (
        <Box
          sx={{
            width: '100%',
            height: '100%',
            display: 'flex',
            flexDirection: 'column',
            p: 2,
          }}
        >
          <Box
            sx={{
              display: 'flex',
              alignItems: 'center',
              mb: 2,
            }}
          >
            <IconButton
              onClick={() => setIsExpanded(false)}
              size="small"
              sx={{
                mr: 1,
                color: 'text.secondary',
                '&:hover': {
                  color: 'primary.main',
                },
              }}
            >
              <ArrowForwardIosIcon sx={{ fontSize: 18 }} />
            </IconButton>
            <Typography variant="h6">
              Available Widgets
            </Typography>
          </Box>

          <List
            sx={{
              flex: 1,
              overflowY: 'auto',
              '&::-webkit-scrollbar': {
                width: '8px',
              },
              '&::-webkit-scrollbar-track': {
                background: '#f1f1f1',
                borderRadius: '4px',
              },
              '&::-webkit-scrollbar-thumb': {
                background: '#888',
                borderRadius: '4px',
                '&:hover': {
                  background: '#666',
                },
              },
            }}
          >
            {widgets.map((widget) => {
              const isSelected = selectedWidgets.has(widget.type);
              return (
                <ListItem
                  key={widget.type}
                  draggable={!isSelected}
                  onDragStart={(e) => handleDragStart(e, widget.type)}
                  component={Paper}
                  sx={{
                    mb: 2,
                    cursor: isSelected ? 'not-allowed' : 'grab',
                    opacity: isSelected ? 0.5 : 1,
                    backgroundColor: isSelected ? 'action.disabledBackground' : 'background.paper',
                    '&:hover': !isSelected ? {
                      backgroundColor: 'action.hover',
                      transform: 'translateY(-2px)',
                    } : {},
                    transition: 'all 0.2s ease',
                    borderRadius: 1,
                  }}
                >
                  <ListItemIcon sx={{ color: isSelected ? 'action.disabled' : 'primary.main' }}>
                    {widget.icon}
                  </ListItemIcon>
                  <ListItemText
                    primary={widget.label}
                    secondary={isSelected ? 'Already added to dashboard' : widget.description}
                    primaryTypographyProps={{
                      fontWeight: 'medium',
                      color: isSelected ? 'text.disabled' : 'text.primary'
                    }}
                  />
                </ListItem>
              );
            })}
          </List>
        </Box>
      ) : (
        <Box
          sx={{
            width: '100%',
            height: '100%',
            display: 'flex',
            flexDirection: 'column',
            alignItems: 'center',
            backgroundColor: 'background.paper',
            pt: 2,
            pb: 2,
          }}
        >
          <IconButton
            onClick={() => setIsExpanded(true)}
            size="small"
            sx={{
              mb: 3,
              color: 'text.secondary',
              '&:hover': {
                color: 'primary.main',
              },
            }}
          >
            <ArrowBackIosNewIcon sx={{ fontSize: 20 }} />
          </IconButton>

          <Box
            sx={{
              display: 'flex',
              flexDirection: 'column',
              gap: 2,
              alignItems: 'center',
            }}
          >
            {widgets.map((widget) => {
              const isSelected = selectedWidgets.has(widget.type);
              return (
                <Tooltip key={widget.type} title={widget.label} placement="left">
                  <Paper
                    draggable={!isSelected}
                    onDragStart={(e) => handleDragStart(e, widget.type)}
                    sx={{
                      p: 1,
                      width: 40,
                      height: 40,
                      cursor: isSelected ? 'not-allowed' : 'grab',
                      opacity: isSelected ? 0.5 : 1,
                      backgroundColor: isSelected ? 'action.disabledBackground' : 'background.paper',
                      '&:hover': !isSelected ? {
                        backgroundColor: 'action.hover',
                        transform: 'translateX(-2px)',
                      } : {},
                      transition: 'all 0.2s ease',
                      display: 'flex',
                      alignItems: 'center',
                      justifyContent: 'center',
                    }}
                  >
                    <Box 
                      sx={{ 
                        color: isSelected ? 'action.disabled' : 'primary.main',
                        display: 'flex',
                        '& > svg': {
                          fontSize: 24,
                        }
                      }}
                    >
                      {widget.icon}
                    </Box>
                  </Paper>
                </Tooltip>
              );
            })}
          </Box>
        </Box>
      )}
    </Box>
  );
};

export default Settings;
```
